## Forcing files found in .res to Be Loaded First | or Only Loaded
_To make it load first, move this before the last return false._
_To make files not load unless they are in a .res file, delete instead_

- **CResFile::Open**
	```cpp
	if (CFileIO::Open(lpszFileName, mode))
    {
	#ifdef __SECURITY_0628
        CString strFileName = lpszFileName;
        if (strFileName.Find("\\", 0) < 0)
        {
            ::Error("killed by CResFile::Open() %s, %s, 2", prj.GetText(TID_GAME_RESOURCE_MODIFIED), lpszFileName);
            ExitProcess(-1);
        }
	#endif    
        return true;
    }
	```
