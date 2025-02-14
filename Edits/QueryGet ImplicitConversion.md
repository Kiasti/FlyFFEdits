## Implicit Conversion for CQuery
---
The CQuery class manages connecting to a database, and running / returning the query results. When calling the execute function within the CQuery class, it prepares the fetch by binding 256 char*'s to the sql results `SQLBindCol(hStmt, c + 1, SQL_C_CHAR, Col[c], 8192, &lCol[c]);` and at the same time, it make sure to bind the metadata `SQLDescribeCol(hStmt,c+1,ColName[c],30, &nActualLen, &m_nSQLType, &m_nPrecision, &m_nScale, &m_nNullability);`

The mssql connector / odbc driver will handle data conversion from the type to the char*. (memory allocation?). Then the client converts it back to the desired value. This isn't really ideal, as it can cause much extra unneeded overhead, but it is more of a universal approach for any interaction that you will need with the DB and the CQeury class. 

> Note: <small>The SQLDescribeCol function is able to return the column type, but it only retrieves and uses the column name in the flyff source, and that leads towards more dynamic binding of parameters.</small>

Because the data is converted to char*, the "client" (databaseserver), needs to reconvert the data back or "interpret" the data. 

In FlyFF's implementation, they have multiple functions designed to convert and return the data type within the class, (E.g. `GetInt(char*), GetFloat(char*)` running a look up on the stored Column Names then `GetInt(int)` to convert the data.

With C++Newest enabled, there are compilation errors due to more type safety working with string literalls. Take the following code as an example: `qry->GetInt("ColumnName");`, would then compile in to an error because "ColumnName" is considered a constant string literal and thus, the implementation of `GetInt(char*)` becomes `GetInt(const char*)`. 

I thought "instead of changing every function, why not write an implicit converter for CQuery class". This way, the type would be reliant on the variable that it is being set to, because the data buffers is stored as a char* anyway, and binds the char* to the sql.


Let's take a look at the full function:

```cpp
int CQuery::GetInt(int nCol)
{
	if (nCol > this->nCol)
		return CQUERYNOCOL;

	if (lCol[nCol-1]==SQL_NULL_DATA)
		return CQUERYNULL;
	return atoi(Col[nCol-1]);
}

int CQuery::GetInt(const char *sCol)
{
	int n;
	n=FindCol(sCol);
	if (n==-1)
		return CQUERYNOCOL;
	return GetInt(n);
}
```

The first step is finding the column. If there isn't a column, it then returns that value. Likewise when the collumn is a nulld data type.

---


To implcitly convert, instead of returning the actual type (E.g `int, float, double`) we need to return a custom struct. We need the column where the value is stored, and we need to conver the data in the custom struct.
```cpp
	struct RetTypes
	{
		const char* ColValue{ nullptr };
		RetTypes(const char* colval) : ColValue(colval) { }
		[[nodiscard]] operator int() const {
			const int value = std::atoi(ColValue); 
			return value;
		}
	}
```

Then we just need one simple get function:
```cpp
		RetTypes Get(const char* str) const
		{
			const int columnIndex = FindCol(str);
			return RetTypes(str);
		}
```


From the original code, there are still things that need to be implemented, like all the errors. When columnIndex is -1 or when it is greater than the maximum column, `CQUERYNOCOL` is returned, and this happens for every type BUT strings.

```cpp
	enum { CQUERYNULL=-100, CQUERYEOF=-101, CQUERYNOCOL=-102, CQUERYERROR=-103 };
```

Let's update RetTypes for error handling, and that data needs to enter RetTypes.

```cpp
	struct RetTypes
	{
		const char* ColValue{ nullptr };
		int error_code{ 0 }; 
		
		explicit RetTypes(const int errorCode, const char* val) : ColValu(colval), error_code(errorCode)
		{
		}

		RetTypes(const char* colval) : ColValue(colval) { }
		[[nodiscard]] operator int() const {
			const int value = error_code != 0 ? error_code : std::atoi(ColValue); 
			return value;
		}
	}

	RetTypes Get(const char* str) const
	{
		const int columnIndex = FindCol(str);
		if (columnIndex == -1 || columnIndex > this->nCol)
			return RetTypes(CQUERYNOCOL, str);
		if (lCol[columnIndex - 1] == SQL_NULL_DATA)
			return RetTypes(CQUERYNULL, str);
		return RetTypes(str, Col[columnIndex - 1]);
	}
```

That code should work, but is it ideal? What if there is an incorrect column, or conversion? Where is the logging? There isn't any. `atoi` is deprecated, and if it tried to convert a a string that isn't a number, it'll still return 0. Otherwise, take the following string `  123ganjawhitenight`. The function would return 123. Both functions would end up ignoring the whitespace of course. Let's replace the use of `atoi` to a safer alternative, and start to handle errors.


```cpp

	struct RetTypes
	{
		const char* ColumnName{ nullptr };
		const char* ColValue{ nullptr };
		int error_code{ 0 };

		RetTypes(const char* Name, const char* var) : ColumnName(Name), ColValue(var) { }
		explicit RetTypes(const int errorCode, const char* Name) {
			ColumnName = Name;
			error_code = errorCode;
			DumpError();
		}

		void DumpError() const { Error("[Query] ColName, Value: %s, %s, %d", ColumnName, ColValue == nullptr ? "NULL" : ColValue, error_code); }

		[[nodiscard]] const char* GetStringError() const
		{
			switch (error_code)
			{
				case CQUERYNOCOL:
					return "NOCOL";
				case CQUERYNULL:
					return "NULL";
				case CQUERYEOF:
					return "EOF";
				case CQUERYERROR:
				default:
					return "UNKNOWN ERROR";
			}
		}

		bool CheckConversion(const char* endPtr)
		{
			if (*endPtr != '\0' || endPtr == ColValue) {
				error_code = CQUERYERROR;
				DumpError();
				return false;
			}
			return true;
		}
		[[nodiscard]] operator long()
		{
			if (error_code < 0)
				return error_code;

			char* endPtr{ nullptr };
			const auto val = strtol(ColValue, &endPtr, 10);
			return CheckConversion(endPtr) ? val : error_code;
		}
		[[nodiscard]] operator unsigned long() 		
		{ 
			if (error_code < 0)
				return error_code;
		
			char* endPtr{ nullptr };
			const auto val = strtoul(ColValue, &endPtr, 10);
			return CheckConversion(endPtr) ? val : error_code;
		}
		[[nodiscard]] operator long long()
		{
			if (error_code < 0)
				return error_code;
		
			char* endPtr{ nullptr };
			const auto val = strtoll(ColValue, &endPtr, 10);
			return CheckConversion(endPtr) ? val : error_code;
		}
		[[nodiscard]] operator unsigned long long()
		{
			if (error_code < 0)
				return error_code;
		
			char* endPtr{ nullptr };
			const auto val = strtoull(ColValue, &endPtr, 10);
			return CheckConversion(endPtr) ? val : error_code;
		}
		[[nodiscard]] operator double()
		{
			if (error_code < 0)
				return error_code;

			char* endPtr{ nullptr };
			const auto val = strtod(ColValue, &endPtr);
			return CheckConversion(endPtr) ? val : static_cast<double>(error_code);
		}
		[[nodiscard]] operator float()
		{
			if (error_code < 0)
				return static_cast<float>(error_code);

			char* endPtr{ nullptr };
			const auto val = strtof(ColValue, &endPtr);
			return CheckConversion(endPtr) ? val : static_cast<float>(error_code);
		}
		[[nodiscard]] operator std::string() const
		{
			return error_code < 0 ? GetStringError() : ColValue;
		}

		[[nodiscard]] operator const char* () const
		{
			return error_code < 0 ? GetStringError() : ColValue;
		}

		[[nodiscard]] operator bool()
		{
			if (error_code < 0)
				return false;

			if ((ColValue[0] | 0x20) == 't')
				return true;

			return static_cast<bool>(operator int()) != 0;
		}
	};
```

Lots of code duplication so far. We can probably reduce that now. One way is using templating, which for this, I wouldn't recommend. That's kinda going away from the implicit conversion, then depending on what you'd template, you'd need to use requires or static_assert (E.g. <small>if you were to pass in the function, you'd need to determine if its invocable and then...</small>), it wouldn't really be worth. 

The other way, is the old fashioned C way of using macros. 

```cpp
		#define DEFINE_INTEGER_CONVERSION(type, conv_fn)				\
		[[nodiscard]] operator type() noexcept							\
		{																\
			if (error_code < 0)											\
				return static_cast<type>(error_code);					\
																		\
			char* end{};												\
			type val = static_cast<type>(conv_fn(ColValue, &end, 10));	\
			if (CheckConversion(end))									\
				return static_cast<type>(error_code);					\
			return val;													\
		}
		DEFINE_INTEGER_CONVERSION(long, strtol);
		DEFINE_INTEGER_CONVERSION(unsigned long, strtoul);
		DEFINE_INTEGER_CONVERSION(long long, strtoll);
		DEFINE_INTEGER_CONVERSION(unsigned long long, strtoull);

```

Now, we need to create a function within the CQuery class to use to return the custom made struct and utilize the implicit conversion.

```cpp
		RetTypes Get(const char* columname) const
		{
			const int columnIndex = FindCol(columname);
			if (columnIndex == -1)
				return RetTypes(CQUERYNOCOL, columname);
			if (columnIndex > this->nCol)
				return RetTypes(CQUERYNOCOL, columname);
			if (lCol[columnIndex - 1] == SQL_NULL_DATA)
				return RetTypes(CQUERYNULL, columname);

			return RetTypes(columname, Col[columnIndex - 1]);
		}
```

Looks good right? well... Not really. Not just yet.

Essentially, every time Get is called, we create a temporary. Normally compilers can optimize them out, but that is highly depending on the compiler settings (E.g <small>Optimizations, debug, etc</ssmall>)

Let's think about a getter struct, one that is a member of the CQuery class that returns a reference -- and that Getter struct will handle the CQuery object getting data, and a stored RetTypes variable.


**Stack Behavior Analysis** |
| Approach	| Temporary Objects	| Getter Struct |
| -------- 	| -----------  | ------ |
| Per-Call Stack Growth	| sizeof(RetTypes) per call (24B)	| Fixed 24B buffer (no growth) |
| Compiler Optimization | RVO may reuse stack slot	| Guaranteed reuse (no RVO needed) |
| 10 Calls in Sequence | 240B stack (if no RVO) → RISKY	| 24B total → Safe |
| Debug Builds | No RVO → Full copies | Same as release → Safe |


```cpp

	struct Getter 
	{
		CQuery* qry;
		RetTypes ret;
		
		Getter(CQuery& query) { qry = &query; }
		Getter(CQuery* query) { qry = query; }

		const RetTypes& Get(const char* str)
		{
			if (const int columnIndex = qry->FindCol(str); columnIndex == -1 || columnIndex > qry->nCol)
				ret = RetTypes(CQUERYNOCOL, str );
			else if (qry->lCol[columnIndex - 1] == SQL_NULL_DATA)
				ret = RetTypes(CQUERYNULL, str);
			else
				ret = RetTypes(str, qry->Col[columnIndex - 1]);
			return ret;
		}

		template <size_t N>
		void Get(const char* str, char(&a)[N])
		{
			strcpy_s(a, Get(str));
		}
		
		void Get(const char* str, char* out, const size_t len)
		{
			strcpy_s(out, len, Get(str));
		}
	};
	Getter getter_{this};
```

---

We can also make sure the compiler force inlines the conversions, and utilize branch predictability hinting to make the code slightly more quick because the likeliness of encountering and error, isn't very likely.


```cpp
#ifdef _MSC_VER
#define UNLIKELY(expr) (expr) [[unlikely]]
#define LIKELY(expr) (expr) [[likely]]
#else
#define UNLIKELY(expr) (__builtin_expect(!!(expr), 0))
#define LIKELY(expr) (__builtin_expect(!!(expr), 1))
#endif
```

---

**Here is the full code all together**


```cpp
// Above CQuery class or in a helper file.
#ifdef _MSC_VER // in C++20, the keywords are added.
#define UNLIKELY(expr) (expr) [[unlikely]]
#define LIKELY(expr) (expr) [[likely]]
#else 
#define UNLIKELY(expr) (__builtin_expect(!!(expr), 0))
#define LIKELY(expr) (__builtin_expect(!!(expr), 1))
#endif
```

```cpp
// Either inside or outside the CQuery class. Could make a separate file.
struct RetTypes
{
	const char* ColumnName{ nullptr };
	const char* ColValue{ nullptr };
	int error_code{ 0 };

	RetTypes() = default;
	RetTypes(const char* Name, const char* var) : ColumnName(Name), ColValue(var) { }
	explicit RetTypes(const int errorCode, const char* Name) {
		ColumnName = Name;
		error_code = errorCode;
		DumpError();
	}

	void DumpError() const { Error("[Query] ColName, Value: %s, %s, %d", ColumnName, ColValue == nullptr ? "NULL" : ColValue, error_code); }

	[[nodiscard]] const char* GetStringError() const
	{
		switch (error_code)
		{
			case CQUERYNOCOL:
				return "NOCOL";
			case CQUERYNULL:
				return "NULL";
			case CQUERYEOF:
				return "EOF";
			case CQUERYERROR:
			default:
				return "UNKNOWN ERROR";
		}
	}

	[[nodiscard]] __forceinline bool CheckConversion(const char* end) noexcept
	{
		if UNLIKELY(end == ColValue || *end != '\0')
		{
			error_code = CQUERYERROR;
			DumpError();
			return false;
		}
		return true;
	}

	#define DEFINE_INTEGER_CONVERSION(type, conv_fn)				\
	[[nodiscard]] __forceinline operator type() noexcept			\
	{																\
		if UNLIKELY(error_code < 0)									\
			return static_cast<type>(error_code);					\
																	\
		char* end{};												\
		type val = static_cast<type>(conv_fn(ColValue, &end, 10));	\
		if (CheckConversion(end))									\
			return static_cast<type>(error_code);					\
		return val;													\
	}
	#define DEFINE_FLOAT_CONVERSION(type, conv_fn)					\
	[[nodiscard]] __forceinline operator type() noexcept			\
	{																\
		if UNLIKELY(error_code < 0)									\
			return static_cast<type>(error_code);					\
																	\
		char* end{};												\
		type val = static_cast<type>(conv_fn(ColValue, &end));		\
		if (CheckConversion(end))									\
			return static_cast<type>(val);							\
		return val;													\
	}

	DEFINE_INTEGER_CONVERSION(long, strtol);
	DEFINE_INTEGER_CONVERSION(unsigned long, strtoul);
	DEFINE_INTEGER_CONVERSION(long long, strtoll);
	DEFINE_INTEGER_CONVERSION(unsigned long long, strtoull);
	DEFINE_FLOAT_CONVERSION(double, strtod);
	DEFINE_FLOAT_CONVERSION(float, strtof);
	#undef DEFINE_INTEGER_CONVERSION
	#undef DEFINE_FLOAT_CONVERSION
	
	[[nodiscard]] __forceinline operator bool()
	{
		if (error_code < 0)
			return false;

		if ((ColValue[0] | 0x20) == 't')
			return true;

		return operator int() != 0;
	}

	[[nodiscard]] __forceinline operator std::string() const noexcept { return error_code < 0 ? GetStringError() : ColValue; }
	[[nodiscard]] __forceinline operator const char* () const noexcept { return error_code < 0 ? GetStringError() : ColValue; }
	[[nodiscard]] __forceinline operator char() noexcept { return static_cast<char>(operator long()); }
	[[nodiscard]] __forceinline operator unsigned char() noexcept { return static_cast<unsigned char>(operator long()); }
	[[nodiscard]] __forceinline operator short() noexcept { return static_cast<short>(operator long()); }
	[[nodiscard]] __forceinline operator unsigned short() noexcept { return static_cast<unsigned short>(operator long()); }
	[[nodiscard]] __forceinline operator int() noexcept { return  operator long(); }
};
```

```cpp
// Either inside CQuery class, or another file
struct Getter 
{
	CQuery* qry;
	RetTypes ret;
	
	Getter(CQuery& query) { qry = &query; }
	Getter(CQuery* query) { qry = query; }
	RetTypes& Get(const char* str)
	{
		if (const int columnIndex = qry->FindCol(str); columnIndex == -1 || columnIndex > qry->nCol)
			ret = RetTypes(CQUERYNOCOL, str );
		else if (qry->lCol[columnIndex - 1] == SQL_NULL_DATA)
			ret = RetTypes(CQUERYNULL, str);
		else
			ret = RetTypes(str, qry->Col[columnIndex - 1]);
		return ret;
	}
	template <size_t N>
	void Get(const char* str, char(&a)[N])
	{
		strcpy_s(a, Get(str));
	}
	
	void Get(const char* str, char* out, const size_t len)
	{
		strcpy_s(out, len, Get(str));
	}
};
```

	
```cpp
	// Add the variable to the CQuery class.
	Getter getter_{this};
```

<br>

**Usage**:
```cpp
	//CQuery::Getter getter(pQuery); // You can create a separate one
	CQuery::Getter& getter = pQuery->getter_; // or you can tie it back to the variable stored in CQuery.
	// In shorter queries, it'd be acceptable to do pQuery->getter_.Get()
	if (pQuery->Execute("usp_GuildHouse_MaxSEQ '%02d'", g_appInfo.dwSys))
	{
		if (pQuery->Fetch())
			m_nSeqNum = getter.Get("SEQ");
	}
	else
		return FALSE;

	if (pQuery->Execute("usp_GuildHouse_Load '%02d'", g_appInfo.dwSys))
	{
		while (pQuery->Fetch())
		{
			const DWORD dwGuildId = getter.Get("m_idGuild");
			const DWORD dwWorldId = getter.Get("dwWorldID");
			const time_t tKeepTime = getter.Get("tKeepTime");

```


**Built Databaseserver.exe**
```cpp
========== Build: 1 succeeded, 0 failed, 0 up-to-date, 0 skipped ==========
```

> Note: <small> Code needs to be tested for any inconsistencies or issues.</small>

<br>
<br>

## License
---
_FlyFF is an mmorpg created by GalaNet/Aeonsoft. This implementation was done by myself:_
| Account  | Site |
| -------- | ------- |
| Kiaemi. | Discord |
| Velvet | RageZone |
| M¿dScientist | Elitepvpers |


