## Mob Element Display
---
- _This will display the weak against and strong against element for that mob_
[![Element Tooltip](http://img.youtube.com/vi/lOiMsPm4bTY/0.jpg)](https://www.youtube.com/watch?v=lOiMsPm4bTY))


## Adding Parts to Source
---
- **Put in PutToolTip / PutToolTipEx Functions**
	```cpp
		#ifdef __MobElementDisplay
			mobElement = 0;
		#endif
	```
- **Data.h**  
    Within: **namespace SAI79** after the enum.  
    _Create some inline functions in the same namespace as elements._  
    ```cpp
    #ifdef __MobElementDisplay
	    inline ePropType GetWeakAgainst(ePropType ep)
	    {
	    	switch (ep)
	    	{
		    	case FIRE: return WATER;
			    case WATER: return ELECTRICITY;
			    case ELECTRICITY: return EARTH;
			    case WIND: return FIRE;
			    case EARTH: return WIND;
			    default: 
		    		return NO_PROP;
		    }
	    }
	    inline ePropType GetStrongAgainst(ePropType ep)
	    {
		    switch (ep)
		    {
			    case FIRE: return WIND;
			    case WATER: return FIRE;
			    case ELECTRICITY: return WATER;
			    case WIND: return EARTH;
			    case EARTH: return ELECTRICITY;
			    default: return NO_PROP;
		    }
	    }
    #endif
    ```
- **Tooltip.h**  
    _Create the variable to store mob element and to set the variable_  
    in: **class CToolTip** before public delcaration  
    ```CPP
    #ifdef __MobElementDisplay
    	char mobElement = 0;
    #endif
    ```
    After public delcaration:
    ```cpp
    #ifdef __MobElementDisplay
    	void SetMonsterElement(const SAI79::ePropType e) { mobElement = static_cast<char>(e); }
    #endif
    ```

- **Tooltip.cpp**  
    in: **CToolTip::Paint(C2DRender* p2DRender)**  
    Add inbetween this endif and if statement for this result:  
    ```cpp
        #endif // __IMPROVE_MAP_SYSTEM


        #ifdef __MobElementDisplay
		if (mobElement && mobElement < SAI79::ePropType::END_PROP)
		{
			int nMonElementYPos = 0;
			int nStringLine = 0;
			for (int i = 0; i < 2; ++i)
			{
				CString strTemp = m_strToolTip.GetLine(nStringLine);
				CString strEnd = strTemp.GetAt(strTemp.GetLength() - 1);

				int nLine = 1;
				if (strEnd != "\n")
				{
					nLine = 2;
					++nStringLine;
					strTemp = m_strToolTip.GetLine(nStringLine);
				}

				strTemp.TrimRight();
				CSize size = CWndBase::m_Theme.m_pFontText->GetTextExtent(strTemp);

				if (i == 0)
				{
					nMonElementYPos = PlusRect.top + 8 + (size.cy + 2) * (nLine - 1);
					g_WndMng.m_pWndWorld->m_texAttrIcon.Render(p2DRender, CPoint(PlusRect.left + size.cx + 15, nMonElementYPos), SAI79::GetWeakAgainst(static_cast<SAI79::ePropType>(mobElement)) - 1, 255, 1.0f, 1.0f);
				}
				else
				{
					nMonElementYPos += 2 + (size.cy + 2) * nLine;
					g_WndMng.m_pWndWorld->m_texAttrIcon.Render(p2DRender, CPoint(PlusRect.left + size.cx + 13, nMonElementYPos), SAI79::GetStrongAgainst(static_cast<SAI79::ePropType>(mobElement)) - 1, 255, 1.0f, 1.0f);
				}
				++nStringLine;
			}
		}
        #endif

    #if __VER >= 9 // __CSC_VER9_1
		if(m_nAdded == 1)
		{
    ```
    
- **WndWorld.cpp**  
   _On Hover, do the tool tip :)_
    ```CPP
    void CWndWorld::OnMouseWndSurface( CPoint point )
    {
    #ifdef __MobElementDisplay
    	CWorld* const pWorld = g_WorldMng.Get();
    
    	if (pWorld)
    	{
    		CMover* pMover = dynamic_cast<CMover*>(pWorld->GetObjFocus());
    		if (pMover)
    		{
    			const MoverProp* const mProp = pMover->GetProp();
    			if (mProp)
    			{
    				const int nPos = (GetClientRect().Width() - 200) / 2;
    				CRect fakeRect = CRect(nPos, 5, nPos + 32, 37);
    				if (fakeRect.PtInRect(point))
    				{
    					ClientToScreen(&point);
    					ClientToScreen(&fakeRect);
    
    					CString tempStr = "Weakness: \n";
    					CEditString editStr = _T("");
    
    					editStr.AddString(tempStr, 0xFF000000);
    					tempStr = "Resistance: \n";
    					editStr.AddString(tempStr, 0xFF000000);
    
    					g_toolTip.PutToolTip(99009933, editStr, *fakeRect, point, 2);
    					g_toolTip.SetMonsterElement(mProp->eElementType);
    				}
    			}
    		}
    	}
    #endif
    }
    ```
    

## Thank you
---
_FlyFF is an mmorpg created by GalaNet/Aeonsoft. This implementation was done by myself:_
| Account  | Site |
| -------- | ------- |
| Kiaemi. | Discord |
| Velvet | RageZone |
| M¿dScientist | Elitepvpers |

