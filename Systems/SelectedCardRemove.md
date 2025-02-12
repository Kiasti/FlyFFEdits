## Selected Card Remove + Ultimate Remove
---
You can select the card you want to remove with this feature.


- DpClient.cpp
```cpp
#ifdef __SelectedCardRemove
void CDPClient::SendPiercingRemove(unsigned long objid, unsigned long pos)
{
	BEFORESENDSOLE(ar, PACKETTYPE_PIERCINGREMOVE, DPID_UNKNOWN);
	ar << objid << pos;
	SEND(ar, this, DPID_SERVERPLAYER);
}
#else
void CDPClient::SendPiercingRemove( unsigned long objid )
{
	BEFORESENDSOLE( ar, PACKETTYPE_PIERCINGREMOVE, DPID_UNKNOWN );
	ar << objid;
	SEND( ar, this, DPID_SERVERPLAYER );
}
#endif
```
```cpp
#ifdef __SelectedCardRemove
void CDPClient::SendUltimateRemoveGem(unsigned long objItemWeapon, unsigned long objItemGem, unsigned long m_nSelected)
{
	BEFORESENDSOLE(ar, PACKETTYPE_ULTIMATE_REMOVEGEM, DPID_UNKNOWN);
	ar << objItemWeapon;
	ar << objItemGem;
	ar << m_nSelected;
	SEND(ar, this, DPID_SERVERPLAYER);
}
#else
void CDPClient::SendUltimateRemoveGem(unsigned long objItemWeapon, unsigned long objItemGem)
{
	BEFORESENDSOLE(ar, PACKETTYPE_ULTIMATE_REMOVEGEM, DPID_UNKNOWN);
	ar << objItemWeapon;
	ar << objItemGem;
	SEND(ar, this, DPID_SERVERPLAYER);
}
#endif
```

- DPClient.h

```cpp
#ifdef __SelectedCardRemove
	void SendPiercingRemove(unsigned long objid, unsigned long pos);
#else
	void	SendPiercingRemove(unsigned long objid);
#endif
```

```cpp
#ifdef __SelectedCardRemove
	void SendUltimateRemoveGem(unsigned long objItemWeapon, unsigned long objItemGem, unsigned long m_nSelected);
#else
	void SendUltimateRemoveGem(unsigned long objItemWeapon, unsigned long objItemGem);
#endif
```



- DPSrvr.cpp
<small>*CDPSrvr::OnPiercingRemove*</small>

```cpp
#ifdef __SelectedCardRemove
	unsigned long pos;
	ar >> pos;
#endif

	CUser* pUser = g_UserMng.GetUser( dpidCache, dpidUser );
	if( !IsValidObj( pUser ) )
		return;

#ifdef __SelectedCardRemove
	CItemUpgrade::GetInstance()->OnPiercingRemove(pUser, objId, pos);
#else
	CItemUpgrade::GetInstance()->OnPiercingRemove( pUser, objId);
#endif
```

<small>*CDPSrvr::OnUltimateRemoveGem*</small>

```cpp
#ifdef __SelectedCardRemove
	unsigned long currentSelection;
	ar >> currentSelection;

	int nResult = prj.m_UltimateWeapon.RemoveGem(pUser, objItemWeapon, objItemGem, currentSelection);
#else
	int nResult = prj.m_UltimateWeapon.RemoveGem(pUser, objItemWeapon, objItemGem);
#endif
```


- ItemUpgrade.cpp

<small>*CItemUpgrade::OnPiercingRemove*</small>

```cpp
#ifdef __SelectedCardRemove
void CItemUpgrade::OnPiercingRemove(CUser* pUser, DWORD objId, unsigned long pos)
#else
void CItemUpgrade::OnPiercingRemove( CUser* pUser, DWORD objId )
#endif

// in the same func:


#ifdef __SelectedCardRemove
	if (pos >= pItemElem->GetPiercingSize())
		return;
#endif

	int nPayPenya = 1000000; // ÁöºÒÇÒ Æä³Ä

// in the same func again

#ifdef __SelectedCardRemove
	if (pItemElem->GetPiercingItem(pos) != 0)
	{
		pUser->AddGold(-nPayPenya);	// Æä³Ä ÁöºÒ
		pUser->AddDefinedText(TID_GAME_REMOVE_PIERCING_SUCCESS);
#ifdef __fixes 
		CItemElem* visPet = pUser->GetVisPetItem();
		if (visPet && visPet->m_dwObjId == pItemElem->m_dwObjId)
		{
			pUser->ResetPetVisDST(visPet);
			pUser->UpdateItem(static_cast<unsigned char>(pItemElem->m_dwObjId), UI_PIERCING, MAKELONG(pos, 0));
			pUser->SetPetVisDST(visPet);
		}
		else
			pUser->UpdateItem(static_cast<unsigned char>(pItemElem->m_dwObjId), UI_PIERCING, MAKELONG(pos, 0));
#else
		pUser->UpdateItem((BYTE)(pItemElem->m_dwObjId), UI_PIERCING, MAKELONG(pos, 0));
#endif
	}
#else
```



- Piercing.cpp
```cpp
void CPiercing::SetPiercingItem(unsigned long nth, DWORD dwItem )
{
	if( nth >= GetPiercingSize() )
		return;

	m_vPiercing[nth]	= dwItem;
#ifdef __SelectedCardRemove
	if (m_vPiercing[nth] == 0)
		std::rotate(m_vPiercing.begin() + nth, m_vPiercing.begin() + nth + 1, m_vPiercing.end());
#endif
}
```


- UltimateWeapon.cpp
```cpp
#ifdef __SelectedCardRemove
int CUltimateWeapon::RemoveGem(CUser* pUser, OBJID objItemId, OBJID objItemGem, int currentPos)
#else
int CUltimateWeapon::RemoveGem( CUser* pUser, OBJID objItemId, OBJID objItemGem)
#endif

//emited code

#ifdef __SelectedCardRemove
	if (static_cast<unsigned long>(currentPos) >= pItemElem->GetUltimatePiercingSize() || pItemElem->GetUltimatePiercingItem(currentPos) == 0)
#else
	if( pItemElem->GetUltimatePiercingItem( 0 ) == 0 )
#endif

//emited code

	if( nRandom < m_nRemoveGemProb )
	{
#ifdef __SelectedCardRemove
		pUser->UpdateItem((BYTE)(pItemElem->m_dwObjId), UI_ULTIMATE_PIERCING, MAKELONG(currentPos, 0));
#else
```


- UltimateWeapon.h

```cpp
#ifdef __SelectedCardRemove
	int RemoveGem(CUser* pUser, OBJID objItemId, OBJID objItemGem, int pos = -1);
#else
	int RemoveGem( CUser* pUser, OBJID objItemId, OBJID objItemGem );
#endif
```


- WndField.cpp
```cpp
#ifdef __SelectedCardRemove
CWndRemovePiercing::CWndRemovePiercing() : m_nSelected(-1)
#else
CWndRemovePiercing::CWndRemovePiercing() 
#endif
```

<small>*CwndRemovePiercing::OnDraw*</small>

```cpp
#ifdef __SelectedCardRemove
			if (m_nSelected >= 0)
			{
				lpWndCtrl = GetWndCtrl(m_nInfoSlot[m_nSelected]);
				p2DRender->RenderFillRect(lpWndCtrl->rect, 0x60FFFF00);//
				p2DRender->RenderRect(lpWndCtrl->rect, 0xFF46C343);
			}
#endif
			for(int i=0; i<nPiercingSize; i++)
			{
				if(nPiercingSize > nMaxPiercing)
					break;

				PPIERCINGAVAIL ptr = NULL;
				ptr = CPiercingAvail::GetInstance()->GetPiercingAvail( m_pItemElem->GetPiercingItem( i ) );

				if(ptr != NULL)
				{
					for(int j=0; j<ptr->nSize; j++)
					{
						int nDst = (int)ptr->anDstParam[j];
						int nAdj = (int)ptr->anAdjParam[j];
						
						if( g_WndMng.IsDstRate(nDst) )
						{
							if( nDst == DST_ATTACKSPEED )
								strTemp.Format( "%s%+d%%  ", prj.GetText(g_WndMng.GetDSTStringId( nDst )), nAdj / 2 / 10 );
							else
								strTemp.Format( "%s%+d%%  ", prj.GetText(g_WndMng.GetDSTStringId( nDst )), nAdj );
						}
						else
							strTemp.Format( "%s+%d", prj.GetText(g_WndMng.GetDSTStringId( nDst )), nAdj );
						
#ifdef __SelectedCardRemove
						if (j + 1 < ptr->nSize)
							strTemp += ", ";
#endif
						textOpt += strTemp;
					}
					
					lpWndCtrl = GetWndCtrl( m_nInfoSlot[i] );

#ifdef __SelectedCardRemove
					p2DRender->TextOut(lpWndCtrl->rect.left + 6, lpWndCtrl->rect.top + 4, lpWndCtrl->rect.right - 36, lpWndCtrl->rect.bottom, textOpt, D3DCOLOR_ARGB(255, 25, 25, 25));;
#else
					p2DRender->TextOut( lpWndCtrl->rect.left + 10, lpWndCtrl->rect.top + 8, textOpt, D3DCOLOR_ARGB(255,0,0,0) );
#endif
					
					textOpt = "";
					strTemp = "";
				}
			}
```


```cpp

void CWndRemovePiercing::OnLButtonDown( UINT nFlags, CPoint point ) 
{
	if (!m_pItemElem) 
		return;

#ifdef __SelectedCardRemove
	for (int i = 0; i < 10; ++i)
	{
		LPWNDCTRL wndCtrl = GetWndCtrl(m_nInfoSlot[i]);
		if (wndCtrl->rect.PtInRect(point))
		{
			m_nSelected = i;
		}
	}
#endif
}

```

```cpp

void CWndRemovePiercing::OnLButtonDblClk( UINT nFlags, CPoint point )
{
	if(!m_pItemElem) return;
	CRect rect;
	LPWNDCTRL wndCtrl = GetWndCtrl( WIDC_PIC_SLOT );
	rect = wndCtrl->rect;
	if( rect.PtInRect( point ) )
	{
		if(m_pItemElem)
		{
			m_pItemElem->SetExtra(0);
			CWndButton* pButton = (CWndButton*)GetDlgItem(WIDC_START);
			pButton->EnableWindow(FALSE);
			m_pItemElem = NULL;
			m_pEItemProp = NULL;
			m_pTexture = NULL;
		}
	}
#ifdef __SelectedCardRemove
	for (int i = 0; i < 10; ++i)
	{
		wndCtrl = GetWndCtrl(m_nInfoSlot[i]);
		if (wndCtrl->rect.PtInRect(point))
		{
			m_nSelected = i;
			g_DPlay.SendPiercingRemove(m_pItemElem->m_dwObjId, m_nSelected);
		}
	}
#endif
}
```
```cpp
BOOL CWndRemovePiercing::OnChildNotify( UINT message, UINT nID, LRESULT* pLResult ) 
{ 
	if( nID == WIDC_START )
	{
		//¼­¹ö·Î ½ÃÀÛÀ» ¾Ë¸°´Ù.
		if(m_pItemElem != NULL)
		{
#ifdef __SelectedCardRemove
			if (m_nSelected < 0)
				return CWndNeuz::OnChildNotify(message, nID, pLResult);;
#endif
			CWndButton* pButton;
			pButton = (CWndButton*)GetDlgItem( WIDC_START );
			pButton->EnableWindow(FALSE);
#ifdef __SelectedCardRemove
			g_DPlay.SendPiercingRemove(m_pItemElem->m_dwObjId, m_nSelected);
#else
			g_DPlay.SendPiercingRemove(m_pItemElem->m_dwObjId);
#endif
			Destroy();
		}
	}
	return CWndNeuz::OnChildNotify( message, nID, pLResult ); 
} 

```


```cpp
#ifdef __SelectedCardRemove
CWndRemoveJewel::CWndRemoveJewel() : m_nSelected(-1)
#else
CWndRemoveJewel::CWndRemoveJewel() 
#endif
```

(ondraw)
```cpp
			if(m_nJewelID[i] != 0)
			{
				wndCtrl = GetWndCtrl( m_nJewelSlot[i] );

#ifdef __SelectedCardRemove
				if (i == m_nSelected)
				{
					p2DRender->RenderFillRect(wndCtrl->rect, 0x60FFFF00);//
					p2DRender->RenderRect(wndCtrl->rect, 0xFF46C343);
				}
#endif

//emitted code

				wndCtrl = GetWndCtrl( m_nInfoSlot[i] );
#ifdef __SelectedCardRemove
				if (i == m_nSelected)
				{
					p2DRender->RenderFillRect(wndCtrl->rect, 0x60FFFF00);
					p2DRender->RenderRect(wndCtrl->rect, 0xFF46C343);
				}
#endif

```
```cpp
void CWndRemoveJewel::OnLButtonDown( UINT nFlags, CPoint point ) 
{
#ifdef __SelectedCardRemove
	for (int j = 0; j < 2; ++j)
	{	
		int* arrStart = j ? m_nInfoSlot : m_nJewelSlot;
		for (int i = 0; i < 5; ++i)
		{
			LPWNDCTRL wndCtrl = GetWndCtrl(arrStart[i]);
			if (wndCtrl->rect.PtInRect(point))
				m_nSelected = i;
		}
	}
#endif
}
```

```cpp
BOOL CWndRemoveJewel::OnChildNotify( UINT message, UINT nID, LRESULT* pLResult ) 
{ 
	if( nID == WIDC_START )
	{
		//¼­¹ö·Î ½ÃÀÛÀ» ¾Ë¸°´Ù.
		if(m_pItemElem != NULL && m_pMoonstone != NULL)
		{
			CWndButton* pButton;
			pButton = (CWndButton*)GetDlgItem( WIDC_START );
			pButton->EnableWindow(FALSE);

#ifdef __SelectedCardRemove
			if (m_nSelected >= 0)
				g_DPlay.SendUltimateRemoveGem( m_pItemElem->m_dwObjId, m_pMoonstone->m_dwObjId, m_nSelected);
#else
			g_DPlay.SendUltimateRemoveGem(m_pItemElem->m_dwObjId, m_pMoonstone->m_dwObjId);
#endif
			Destroy();
		}
	}
	return CWndNeuz::OnChildNotify( message, nID, pLResult ); 
} 
```

- WndField.h

```cpp
class CWndRemovePiercing : public CWndNeuz
{
#ifdef __SelectedCardRemove
	int m_nSelected;
#endif
```

```cpp
class CWndRemoveJewel : public CWndNeuz
{
#ifdef __SelectedCardRemove
	private:
		int m_nSelected;
#endif

```



--- 
> Note: If you're using Florist's base, then his version should already be included.

## License
---
_FlyFF is an mmorpg created by GalaNet/Aeonsoft. This implementation was done by myself:_
| Account  | Site |
| -------- | ------- |
| Kiaemi. | Discord |
| Velvet | RageZone |
| M¿dScientist | Elitepvpers |

