# Changing Sql Connection to use SQLDriverConnect
- **Query.h**
_Creating an enum and replace Connect method with new method in class CQuery_
  ```cpp
  	enum class SqlDatabase { ACCOUNT, CHARACTER, RANKING, LOGGING, WORLD };
      	bool Connect(SqlDatabase db);
  ```
- Hardcoded strings but this should be swapped out with reading a settings file.
  ```cpp
  constexpr char account[] = "ACCOUNT_DBF";
	constexpr char character[] = "CHARACTER_01_DBF";
	constexpr char logging[] = "LOGGING_01_DBF";
	constexpr char ranking[] = "RANKING_DBF";
	constexpr char username[] = "username(sa)";
	constexpr char server[] = "\\SQLEXPRESS";
	constexpr char password[] = "password";
  ```
- **Query.cpp**
_Method implementation_
  ```cpp
  bool CQuery::Connect(SqlDatabase db)
  {
	  SQLAllocHandle(SQL_HANDLE_ENV, nullptr, &hEnv);
	  SQLSetEnvAttr(hEnv, SQL_ATTR_ODBC_VERSION, reinterpret_cast<SQLPOINTER>(SQL_OV_ODBC3), SQL_IS_INTEGER);
	  SQLAllocHandle(SQL_HANDLE_DBC, hEnv, &hDbc);

    // Insert SQL driver being used with formatting.
	  std::string conStr = Lexer::build("Server=", dbf::server, ";Driver={SQL Server Native Client 11.0};PWD=", dbf::password, ";UID=", dbf::username, ";Database=");
	  switch (db)
	  {
		  case SqlDatabase::ACCOUNT:
			  conStr = Lexer::build(conStr, dbf::account, ";");
			  break;
		  case SqlDatabase::CHARACTER:
			  conStr = Lexer::build(conStr, dbf::character, ";");
			  break;
		  case SqlDatabase::LOGGING:
			  conStr = Lexer::build(conStr, dbf::logging, ";");
			  break;
		  case SqlDatabase::RANKING:
			  conStr = Lexer::build(conStr, dbf::ranking, ";");
			  break;
		  case SqlDatabase::WORLD:
			  conStr = Lexer::build(conStr, c.get("WorldDBName"), ";");
			  break;
		  default:
			  console::echo(Lexer::build("SqlDatabase not configured: ", static_cast<int>(db)), console::note::error);
			  return false;
	  }

	  SQLRETURN ret = SQLDriverConnect(hDbc, nullptr, reinterpret_cast<SQLCHAR*>(const_cast<char*>(conStr.c_str())), SQL_NTS, nullptr, 0,
		nullptr, SQL_DRIVER_NOPROMPT);

	  if (ret != SQL_SUCCESS && ret != SQL_SUCCESS_WITH_INFO)
	  {
		  PrintDiag(conStr.c_str());
		  console::echo(Lexer::build("Could not connect to MSSQL. ", conStr), console::note::error);
		  return false;
	  }
	  ret = SQLAllocHandle(SQL_HANDLE_STMT, hDbc, &hStmt);
	  if ((ret != SQL_SUCCESS) && (ret != SQL_SUCCESS_WITH_INFO))
	  {
		  console::echo("Failed to allocate SQL Handle.", console::note::error);
		  hStmt = nullptr;
		  return false;
	  }
	  return true;
  }
  ```
