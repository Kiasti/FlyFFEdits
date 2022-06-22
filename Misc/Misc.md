# Misc Edits


## Limit X and Y for Text (2drender)
_Automatically scales down font based on restrictions set for x and y._
- **2dRender.cpp**
  ```CPP
    void C2DRender::TextOut(const int x, const int y, const int limitX, const int limitY, const LPCTSTR pszString, const unsigned long dwColor, const unsigned long dwShadowColor)
    {
        if (m_pFont)
        {
            const SIZE size = m_pFont->GetTextExtent(pszString);
            float scale = 1.0f;
            if (size.cx > limitX || size.cy > limitY)
            {
                const float sizeX = static_cast<float>(limitX) / static_cast<float>(size.cx);
                const float sizeY = static_cast<float>(limitY) / static_cast<float>(size.cy);
                scale = sizeX < sizeY ? sizeX : sizeY;
            }
            const CRect rect(x + m_ptOrigin.x, y + m_ptOrigin.y, x + m_ptOrigin.x + size.cx, y + m_ptOrigin.y + size.cy);

            if (m_clipRect.RectLapRect(rect))
            {
                if (dwShadowColor & 0xff000000)m_pFont->DrawText(static_cast<FLOAT>(x + m_ptOrigin.x + 1), static_cast<FLOAT>(y + m_ptOrigin.y), scale, scale, dwShadowColor, const_cast<TCHAR*>(pszString));
                m_pFont->DrawText(static_cast<FLOAT>(x + m_ptOrigin.x), static_cast<FLOAT>(y + m_ptOrigin.y), scale, scale, dwColor, const_cast<TCHAR*>(pszString));
            }
        }
    }
  ```
- **2dRender.h**
  ```CPP
    void TextOut(int x, int y, int limitX, int limitY, LPCTSTR pszString, unsigned long dwColor, unsigned long dwShadowColor = 0x00000000);
  ```
  
## Critical Heal Idea
_Gives a chance on player healing for the healing to be doubled._
- **MoverActEvent.cpp** - CMover::ApplyParam
  ```cpp
  				int nIncHP = pAddSkillProp->nAdjParamVal1 + dwHp1 + dwHp2;
					nIncHP += ((CMover *)pSrc)->GetParam(DST_IMPROVE_HEAL, 0);
					if (nIncHP > 0)
					{
						const int originalVal = nIncHP;
						nIncHP += (int)(originalVal * (((CMover *)pSrc)->GetParam(DST_HEALING_AMP, 0) / 100.f));
						nIncHP += (int)(originalVal * GetParam(DST_RECIEVEHEAL_AMP, 0) / 100.f);
					}
          // __HealingCrits
					float critHeal = 10.0f + (0.03f * static_cast<float>(((CMover*)pSrc)->GetInt()));
					if (critHeal > 90) { critHeal = 90; }
					if (critHeal >= xRandom(0, 100)) { nIncHP = static_cast<int>(static_cast<float>(nIncHP) * 2.0f); }
  ```

## Making Multi-Elemental Skills Get Bonuses From Mastery%
- **MoverAttack.cpp**
  ```CPP
	  case ST_EARTH:	
		  fRatio	= GetParam( DST_MASTRY_EARTH, 0 ) / 100.0f;
		  nATK	= (int)( nATK + (nATK * fRatio) );
		  break;
    // __MultiElementSkillFix
		case ST_FIREEARTH: fRatio = (GetParam(DST_MASTRY_FIRE, 0) / 100.0f) + (GetParam(DST_MASTRY_EARTH, 0) / 100.0f); nATK = (int)(nATK + (nATK * fRatio)); break;
		case ST_EARTHWATER: fRatio = (GetParam(DST_MASTRY_WATER, 0) / 100.0f) + (GetParam(DST_MASTRY_EARTH, 0) / 100.0f); nATK = (int)(nATK + (nATK * fRatio)); break;
		case ST_ELECWIND: fRatio = (GetParam(DST_MASTRY_WIND, 0) / 100.0f) + (GetParam(DST_MASTRY_ELECTRICITY, 0) / 100.0f); nATK = (int)(nATK + (nATK * fRatio)); break;
		case ST_EARTHWIND: fRatio = (GetParam(DST_MASTRY_WIND, 0) / 100.0f) + (GetParam(DST_MASTRY_EARTH, 0) / 100.0f); nATK = (int)(nATK + (nATK * fRatio)); break;
  ```
## Making Non-Elemental Skills Not Get A Bonus On Non-Elemental Item
- **MoverAttack.cpp** : CMover::GetMagicSkillFactor
  ```CPP
  	if (skillType == itemType && skillType != SAI79::NO_PROP)
  ```
## Making DCT Affect all Skills
_By default DCT only affects magic skills_  
- **Mover.cpp** - _CMover::ProcessAniSpeed_
  ```CPP
  	case OBJSTA_ATK_CASTING1:
		case OBJSTA_ATK_CASTING2:
		case OBJSTA_ATK_MAGICSKILL:
    // Adding melee and Range skills to the list.
		case OBJSTA_ATK_MELEESKILL:
		case OBJSTA_ATK_RANGESKILL:
			m_fAniSpeed	= GetCastingAniSpeed();
			break;
  ```
## Cheer Increasing Upgrade Rates Idea
_It's always been a rumor but it was never a thing._
- Example from SmeltAccessory
  ```CPP
  unsigned long dwProbability = CAccessoryProperty::GetInstance()->GetProbability( pItemMain->GetAbilityOption() );
	if (pUser->HasBuff(BUFF_ITEM, II_CHEERUP))
	{
		if (const int temp = static_cast<int>(dwProbability * cheerValueUpgrade); temp < 100) 
			dwProbability += 100; 
		else 
			dwProbability += temp; 
	}
  ```
## Adding an options for Farthest Plane Idea
_This can drastically increase or decrease player land view range._
- **HwOption.cpp**
  _Load_
  ```CPP
		else if (scan.Token == _T("farPlane"))
		{
			CWorld::m_fFarPlane = scan.GetFloat();
		}
  ```
  _Save_
  ```CPP
    _ftprintf(fp, _T("farPlane %f\n"), CWorld::m_fFarPlane);
  ```
## System Messages on Upgrade Max 
- Example
  ```cpp
  #ifdef __SystemMessage_MaxUpgrade
  #include "DPCoreClient.h"
  extern CDPCoreClient g_DPCoreClient;
  #endif
  ```
  ```cpp
		if (pItemMain->GetAbilityOption() + 1 == 10 && !(pUser->IsMode(TRANSPARENT_MODE) && pUser->m_dwAuthorization >= AUTH_GAMEMASTER))
		{
			CString str;
			str.Format("%s: has upgraded [%s] to +10", pUser->GetName(), pItemMain->GetName());
			g_DPCoreClient.SendSystem(str);
		}
  ```
## Monster HP Bars above Model
- CWndWorld::OnEraseBkgnd
  ```cpp
	if( pFocus && pFocus->GetType() == OT_MOVER )
	{
		if( ((CMover *)pFocus)->IsMode( TRANSPARENT_MODE ) )		// »ó´ë°¡ Åõ¸íÈ­ »óÅÂ¸é
			pWorld->SetObjFocus(nullptr );							// Å¸°ÙÀâÀº°Å Ç®¸².
  #ifdef __MonsterHPBars
		else
		{
			if (!dynamic_cast<CMover*>(pFocus)->IsPeaceful())
			{
				v3PartyMemberDir = dynamic_cast<CMover*>(pFocus)->GetPos() - g_Neuz.m_camera.m_vPos;
				D3DXVec3Normalize(&v3PartyMemberDir, &v3PartyMemberDir);

				if (D3DXVec3Dot(&v3CameraDir, &v3PartyMemberDir) >= 0.0f)
					dynamic_cast<CMover*>(pFocus)->RenderHP(g_Neuz.m_pd3dDevice);
			}
		}
  #endif
  ```
  
## Shift to See Set Effects
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
## Forcing files found in .res to Be Loaded First | or Only Loaded
_To make it load first, move this before the last return false, otherwise delete_
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
## Small Optimzation with Inventories
- **CWndItemCtrl::OnDraw**  
	_either undefine or remove_
	```cpp
	#ifndef __fixes
		int nRange = m_pItemContainer->m_dwIndexNum / nWidth;// - nPage;
		if( m_pItemContainer->m_dwIndexNum % nWidth )
			nRange++;
		m_wndScrollBar.SetScrollRange( 0, nRange );
		m_wndScrollBar.SetScrollPage( nPage );

		pt.y = 0;
		pt.y += m_wndScrollBar.GetScrollPos() * nWidth;
	#endif
	```
	_Add with or without ifdefs_
	```cpp
		if( nParent == APP_INVENTORY || nParent == APP_BAG_EX)
		{
	#ifdef __fixes
			int nRange = m_pItemContainer->m_dwIndexNum / nWidth;// - nPage;
			if (m_pItemContainer->m_dwIndexNum % nWidth)
				nRange++;
			m_wndScrollBar.SetScrollRange(0, nRange);
			m_wndScrollBar.SetScrollPage(nPage);
			pt.y = 0;
			pt.y += m_wndScrollBar.GetScrollPos() * nWidth;
			const int maxLoop = pt.y + (nPage * nWidth + nWidth);
	#endif

			for( int i = pt.y; i < m_pItemContainer->GetSize(); i++ ) 
			{
				if( i < 0 ) continue;
	#ifdef __fixes
				if (i > maxLoop)
					break;
	#endif
	```
	_same_
	```cpp
	#ifdef __fixe
			int nRange = m_nArrayCount / nWidth;// - nPage;
			if (m_nArrayCount % nWidth)
				nRange++;
			
			m_wndScrollBar.SetScrollRange(0, nRange);
			m_wndScrollBar.SetScrollPage(nPage);
			
			pt.y = 0;
			pt.y += m_wndScrollBar.GetScrollPos() * nWidth;

			const int maxLoop = pt.y + (nPage * nWidth + nWidth);
			for (int i = pt.y; i < m_nArrayCount; i++)
	#else
			for( int i = 0; i < m_nArrayCount; i++ )
	#endif
			{
	#ifdef __fixes
				if (i < 0 || i > maxLoop)
					break;
	#endif
	```
## List Box Optimization
- **CWndListBox::OnDraw**
	```cpp
		m_nFontHeight = GetFontHeight() + m_nLineSpace;
		{
	#ifndef __fixes
			CPoint pt(3, 3);
			pt.y -= m_nFontHeight * m_wndScrollBar.GetScrollPos();

			PaintListBox(p2DRender, pt, m_listItemArray);
			int nPage = GetClientRect().Height() / m_nFontHeight;
			int nRange = m_listItemArray.GetSize();

			if (IsWndStyle(WBS_VSCROLL))
			{
				m_wndScrollBar.SetVisible(TRUE);
				m_wndScrollBar.SetScrollRange(0, nRange);
				m_wndScrollBar.SetScrollPage(nPage);
			}
			else
				m_wndScrollBar.SetVisible(FALSE);
	#else
			const int nPage = GetClientRect().Height() / m_nFontHeight;
			const int nRange = m_listItemArray.GetSize();

			if (IsWndStyle(WBS_VSCROLL))
			{
				m_wndScrollBar.SetVisible(TRUE);
				m_wndScrollBar.SetScrollRange(0, nRange);
				m_wndScrollBar.SetScrollPage(nPage);
			}
			else
				m_wndScrollBar.SetVisible(FALSE);

			const int nScrollBarWidth = IsWndStyle(WBS_VSCROLL) ? m_wndScrollBar.GetClientRect().Width() : 0;
			CPoint pt(3, 3);
			for (int i = m_wndScrollBar.GetScrollPos(); i < static_cast<unsigned>(m_listItemArray.GetSize()); i++)
			{
				if (i > nPage + m_wndScrollBar.GetScrollPos())
					break;

				tagITEM* pListItem = static_cast<tagITEM*>(m_listItemArray.GetAt(i));
				if (!pListItem->m_bIsVisible)
					continue;

				if (pListItem->m_bIsValid)
				{
					pListItem->m_rect.left = pt.x;
					pListItem->m_rect.top = pt.y;
					pListItem->m_rect.right = pt.x + m_rectWindow.Width() - nScrollBarWidth;
					pListItem->m_rect.bottom = pt.y + m_nFontHeight;

					if (i == m_nCurSelect)
						pListItem->m_strWord.SetColor(m_nSelectColor);
					else
					{
						CPoint point = GetMousePoint();
						if (pListItem->m_rect.PtInRect(point))
							pListItem->m_strWord.SetColor(m_dwOnMouseColor);
						else
							pListItem->m_strWord.SetColor(m_nFontColor);
					}
				}
				else
					pListItem->m_strWord.SetColor(m_dwInvalidColor);
				p2DRender->TextOut_EditString(static_cast<int>(m_nLeftMargin + pt.x), pt.y, pListItem->m_strWord);
				pt.y += m_nFontHeight;
			}
	#endif
	```
## Fix with HP and MP being set when setting a stat point
- **CDPClient::OnSetState**
	_remove as its unneeded_
	```cpp
	g_pPlayer->SetHitPoint( g_pPlayer->GetMaxHitPoint() );
	g_pPlayer->SetManaPoint( g_pPlayer->GetMaxManaPoint() );
	```
## Animation skipping logic frames for dps.
_This i did for someone_
- bone.h
	```cpp
	DWORD IsAttrHit(const float fOldFrm, const float fNumFrm) const
	{ 
	#ifdef __fixes
		unsigned int old = static_cast<unsigned int>(fOldFrm) + 1;
		while (old < static_cast<unsigned int>(fNumFrm))
		{
			MOTION_ATTR *pAttr = &m_pAttr[old];
			if (pAttr->m_dwAttr & MA_HIT)
			{
				if (fOldFrm < pAttr->m_fFrame && pAttr->m_fFrame <= fNumFrm)
					return pAttr->m_dwAttr;
			}
			++old;
		}
	#endif
		MOTION_ATTR	 *pAttr = &m_pAttr[(int)fNumFrm];
		if (pAttr->m_dwAttr & MA_HIT)
		{
			if (fOldFrm < pAttr->m_fFrame && pAttr->m_fFrame <= fNumFrm)	// ÀÌÀü ÇÁ·¹ÀÓÀÌ¶û ÇöÀç ÇÁ·¹ÀÓ »çÀÌ¿¡ Å¸Á¡ÀÌ µé¾îÀÖ¾ú´Â°¡.
				return pAttr->m_dwAttr;
		}
		return 0;
	}
	```
## Thank you




