## Re-enabling YSMOOTH_OBJ
---
This system starts the objects at 0 blending, and then increases the object blend/opacity over time until its full. 


- DPClient.cpp (OnADdObj)
```cpp

#ifdef __YSMOOTH_OBJ
	DWORD dwType = pObj->GetType();
	if( dwType == OT_MOVER )
	{
		pObj->m_bSmooth = true;
	}
#endif 	
	
		
	g_WorldMng.Get()->AddObj( pObj, TRUE );
```

- World3d.cpp
```cpp
					if( m_bMiniMapRender == FALSE && pObj->m_fDistCamera > fDistant[ g_Option.m_nObjectDistant ][ pObj->GetModel()->m_pModelElem->m_dwDistant ] ) 
					{
#ifdef __YSMOOTH_OBJ
						pObj->m_bSmooth = true;
						pObj->m_nSmoothCount = 0;
#endif

						continue;
					}
```
- Obj.h
```cpp
	#ifdef __YSMOOTH_OBJ	
		bool			m_bSmooth{true};
		int				m_nSmoothCount{0};
	#endif
```

Cobj::Render
```cpp

#ifdef __YSMOOTH_OBJ
	if (m_bSmooth && (GetType() == OT_OBJ || GetType() == OT_CTRL))
	{
		if (m_pModel->m_nNoEffect == 2)
			pModel->SetBlendFactor(255);
		else
		{
			m_nSmoothCount += 2;
			if (m_nSmoothCount > 255)
			{
				m_nSmoothCount = 0;
				m_bSmooth = false;
				m_wBlendFactor = 255;
				pModel->SetBlendFactor(m_wBlendFactor);
			}
			else
			{
				pModel->SetBlendFactor(m_nSmoothCount);
			}
		}
	}
#endif
	pModel->Render(pd3dDevice, &m_matWorld);
#endif
```

CMover::Render
```cpp
#ifdef __YSMOOTH_OBJ
		if (m_bSmooth && m_pModel->m_nNoEffect != 2)
		{
			m_nSmoothCount += 2;

			if (m_nSmoothCount > 255)
			{
				m_wBlendFactor = 255;
				m_nSmoothCount = 0;
				m_bSmooth = false;
			}
			else
				m_wBlendFactor = m_nSmoothCount;
		}
#endif 
		m_pModel->SetBlendFactor(m_wBlendFactor);
```
