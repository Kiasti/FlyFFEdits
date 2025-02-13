## AutoHelperThread Stop fix / logout fix
---
Another thing I had to fix from Simple. Since the threads would continue to run while logging out, it can cause issues.

- Neuz.cpp
```cpp
#ifdef __AUTO_HELPER_THREAD
void CNeuzApp::StartAutoHelperThread()
{
	m_AutoHelperThread = AfxBeginThread(AutoHelperThreadLoop, this);

	m_AutoHelperThread->m_bAutoDelete = FALSE;
	m_AutoHelperThread->ResumeThread();
	IsStopping = false;
}

#ifdef __fixes
void CNeuzApp::StopAutoHelperThread() const
{
	m_AutoHelperThread->SuspendThread();
	IsStopping = true;
}
void CNeuzApp::ResumeAutoHelperThread() const
{
	m_AutoHelperThread->ResumeThread();
	IsStopping = false;
}
#endif
```

in CNeuzApp::AutoHelperThreadLoop
```cpp
	while (!IsStopping)
	{		
		// all the code originally
		Sleep(1000);
	}
	Sleep(1000);	
```


- Neuz.h
```cpp
#ifdef __AUTO_HELPER_THREAD
	CWinThread*               	m_AutoHelperThread;
	static UINT					AutoHelperThreadLoop(LPVOID param);
	void						StartAutoHelperThread();
#ifdef __fixes
	void StopAutoHelperThread() const;
	void ResumeAutoHelperThread() const;
	inline static std::atomic<bool> IsStopping { false };
#endif
#endif
```

- WndManager.cpp <small>(CWndMgr::OpenTitle)</small>
```cpp
		Free();
		g_WorldMng.DestroyCurrentWorld();
		g_pPlayer = NULL;
#ifdef __fixes
		g_Neuz.StopAutoHelperThread();
#endif
```

