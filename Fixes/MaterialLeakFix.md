This fix is from AntiMalware or whatever they go by now.


in 
<small>CTextureManager::DeleteMaterial)</small>

```cpp
#ifdef __fixes // _MaterialLeakFix
				if (--m_pMaterial[i].m_nUseCnt == 0)			
#else
				if (m_pMaterial[i].m_nUseCnt == 1)			
#endif
```


