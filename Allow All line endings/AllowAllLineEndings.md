# Allow All Line Endings
_In the v15 leak of the flyff source code, the scanner used to load resource files only checks for \r\n in succession. This just treats each \r and \n as a separate character, treating both as newlines. With this, the person editing the game files does not need to make sure to use windows line endings. File size can also be reduced by switching to mac or unix based line endings._

---
## Adding parts to the source
---
   - **CScanner.cpp**  
     Find and edit: **CScanner::GetLineNum**
     ```CPP
     while (lpProg != pCur)
     {
		if (*pCur == '\r' || *pCur == '\n')
			nLine++;
		pCur++;
     }
     ```
     <br/>Find: **CScanner::GetLastFull**
     ```CPP
     while (iswhite(*m_pProg) && *m_pProg && (*m_pProg != '\r' && *m_pProg != '\n'))
         ++m_pProg;
     char* pCur = token; 
     while (*m_pProg && (*m_pProg != '\r' && *m_pProg != '\n'))
     ```
     <br/>Find: **CScanner::GetToken**
     ```CPP
		if(*m_pProg == '/') 
			{ 
				++m_pProg;
				while ((*m_pProg != '\r' && *m_pProg != '\n') && *m_pProg != '\0') 
					++m_pProg;
				/*if(*m_pProg == '\r') 
					m_pProg+=2;*/
			}
			else if(*m_pProg == '*') 
     ```
     ```CPP
		if(*m_pProg=='"') 
		{
			++m_pProg;
			while (*m_pProg != '"' && (*m_pProg != '\r' && *m_pProg != '\n') && *m_pProg != '\0'){ *pszCur++ = *m_pProg++; }
			++m_pProg; 
			tokenType = STRING; 
		}
		else
		{
			while (*m_pProg != ',' && (*m_pProg != '\r' && *m_pProg != '\n') && *m_pProg != '\0'){ *pszCur++ = *m_pProg++; }
		}
     ```
     ```CPP
		if(*m_pProg=='"') 
		{
			++m_pProg;
			while (*m_pProg != '"' && (*m_pProg != '\r' && *m_pProg != '\n') && *m_pProg != '\0' && (pszCur - token) < MAX_TOKENSTR)
			{
				int count = CopyChar( m_pProg, pszCur );
				m_pProg += count;
				pszCur += count;
			}
     ```
## Thank you
---
_FlyFF is owned and operated and developed by Galanet/Aeonsoft. This is only an implementation made by myself, Kia#8008._

[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen. Thanks SO - http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)

