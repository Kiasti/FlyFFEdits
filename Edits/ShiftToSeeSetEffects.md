  
## Shift to See Set Effects
---
This was the original one I did ages ago. I know there are other's relesed that may be slightly more optimized, but I don't like how the "operate". This one over others will drastically reduce the line count as it wont display parts, or set effects unless Shift is held.

- Project.h
  ```cpp
    typedef struct	__ITEMAVAIL
    {
    	int		nSize;
    	int		anDstParam[MAX_ITEMAVAIL];
    	int		anAdjParam[MAX_ITEMAVAIL];
    	bool bQuip[MAX_ITEMAVAIL]; 
  ```
- WndManager.cpp
  ```cpp
  void CWndMgr::PutSetItemOpt(CMover* pMover, CItemElem* pItemElem, CEditString* pEdit)
  {
	  if (CSetItem* pSetItem = CSetItemFinder::GetInstance()->GetSetItemByItemId(pItemElem->m_dwItemId))
	  {
		  CString strTemp;
		  bool equiped[MAX_HUMAN_PARTS] = {};

		  int nEquiped = 0;
		  for (int i = 0; i < pSetItem->m_nElemSize; i++)
		  {
  			if (CItemElem* item_elem = pMover->GetEquipItem(pSetItem->m_anParts[i]); item_elem && item_elem->m_dwItemId == pSetItem->m_adwItemId[i] && !item_elem->IsFlag(CItemElem::expired))
	  		{
		  		equiped[i] = true;
			  	nEquiped++;
			  }
      }

		  strTemp.Format("\n%s (%d/%d)", pSetItem->GetString(), nEquiped, pSetItem->m_nElemSize);
		  pEdit->AddString(strTemp, dwItemColor[g_Option.m_nToolTipText].dwSetName);

		  if (GetAsyncKeyState(VK_SHIFT) & 0x8000)
		  {
			  for (int i = 0; i < pSetItem->m_nElemSize; i++)
			  {
				  if (ItemProp* pItemProp = prj.GetItemProp(pSetItem->m_adwItemId[i]))
				  {
					  strTemp.Format("\n   %s", pItemProp->szName);
					  if (equiped[i]) { pEdit->AddString(strTemp, dwItemColor[g_Option.m_nToolTipText].dwSetItem1); }
				  	else { pEdit->AddString(strTemp, dwItemColor[g_Option.m_nToolTipText].dwSetItem0); }
			  	}
			  }


			  ITEMAVAIL itemAvail;
			  memset(&itemAvail, 0, sizeof(itemAvail));

			  for (int i = 0; i < pSetItem->m_avail.nSize; i++)
			  {
				  int nFind = -1;
				  for (int j = 0; j < itemAvail.nSize; j++)
				  {
					  if (itemAvail.anDstParam[j] == pSetItem->m_avail.anDstParam[i])
					  {
						  nFind = j;
						  break;
					  }
				  }

				  bool bQuip = true;
				  if (pSetItem->m_avail.anEquiped[i] > nEquiped) { bQuip = false; }

				  if (nFind < 0 || !bQuip) { nFind = itemAvail.nSize++; }
				  itemAvail.bQuip[nFind] = bQuip;
				  itemAvail.anDstParam[nFind] = pSetItem->m_avail.anDstParam[i];
				  itemAvail.anAdjParam[nFind] += pSetItem->m_avail.anAdjParam[i];
			  }

			  for (int i = 0; i < itemAvail.nSize; i++)
			  {
				  const int nDst = itemAvail.anDstParam[i];
				  const int nAdj = itemAvail.anAdjParam[i];

				  if (IsDst_Rate(nDst))
				  {
					  if (nDst == DST_ATTACKSPEED) { strTemp.Format("\n%s: %s% +d%%", prj.GetText(TID_TOOLTIP_SET), FindDstString(nDst), nAdj / 2 / 10); }
					  else { strTemp.Format("\n%s: %s% +d%%", prj.GetText(TID_TOOLTIP_SET), FindDstString(nDst), nAdj); }
				  }
				  else { strTemp.Format("\n%s: %s +%d", prj.GetText(TID_TOOLTIP_SET), FindDstString(nDst), nAdj); }

				  if (!itemAvail.bQuip[i]) { pEdit->AddString(strTemp, 0xEF878887); }
				  else { pEdit->AddString(strTemp, dwItemColor[g_Option.m_nToolTipText].dwSetEffect); }
			  }
		  }
		  else
		  {
			  strTemp.Format("\nHold Shift to see Set Effects");
			  pEdit->AddString(strTemp, dwItemColor[g_Option.m_nToolTipText].dwSetEffect);
	  	}
	  }
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

