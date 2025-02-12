## Buyback
----
- DPClient.cpp
```cpp
#include "Buyback.h"


#ifdef __BUYBACK
		case BuyBack::SNAPSHOTTYPE_BUYBACK:	OnBuyBack(ar); break;
		case BuyBack::SNAPSHOTTYPE_BUYBACK_REMOVE_ITEM:	OnBuyBackRemoveItem(ar); break;
		case BuyBack::SNAPSHOTTYPE_BUYBACK_ALL: OnBuyBackAll(ar); break;
#endif


#ifdef __BUYBACK
void CDPClient::OnBuyBackAll(CAr& ar)
{
	BuyBack::Sys.Serialize(ar);
	if (BuyBack::CWndBuyBack* pWndBuyBack = dynamic_cast<BuyBack::CWndBuyBack*>(g_WndMng.GetWndBase(APP_BUYBACK)))
		pWndBuyBack->ProcessListBox();
}

void CDPClient::OnBuyBack(CAr& ar)
{
	CItemElem itemElem;
	itemElem.Serialize(ar);

	BuyBack::Sys.Add(std::move(itemElem));
	if (BuyBack::CWndBuyBack* pWndBuyBack = dynamic_cast<BuyBack::CWndBuyBack*>(g_WndMng.GetWndBase(APP_BUYBACK)))
		pWndBuyBack->ProcessListBox();
}

void CDPClient::OnBuyBackRemoveItem(CAr& ar)
{
	int nItem;
	ar >> nItem;

	if (nItem != -1)
	{
		BuyBack::Sys.RemoveAtIndex(nItem);
		BuyBack::Sys.Reorganize();

		if (BuyBack::CWndBuyBack* pWndBuyBack = dynamic_cast<BuyBack::CWndBuyBack*>(g_WndMng.GetWndBase(APP_BUYBACK)))
			pWndBuyBack->ProcessListBox();
	}
}

void CDPClient::SendBuyBack(int nItem)
{
	BEFORESENDSOLE(ar, BuyBack::PACKETTYPE_SEND_BUYBACK, DPID_UNKNOWN);
	ar << nItem;
	SEND(ar, this, DPID_SERVERPLAYER);
}
#endif
```

- DPClient.h
```cpp
#ifdef __BUYBACK
		void OnBuyBack(CAr& ar);
		void OnBuyBackRemoveItem(CAr& ar);
		void SendBuyBack(int nItem);
		void OnBuyBackAll(CAr& ar);
#endif
```


- DPDatabaseClient.cpp (Worldserver) onjoin

```cpp
#ifdef __BUYBACK
		if (BuyBack::BuyBackPlayer& bbc = BuyBack::Sys.GetPlayer(pUser->m_idPlayer); bbc.ContainsElements())
			pUser->SendEntireBuyBack(bbc);
#endif
```


- DPSrvr.cpp
```cpp
#ifdef __BUYBACK
	ON_MSG(PACKETTYPE_SEND_BUYBACK, &CDPSrvr::OnBuyBack);
#endif
```


CDPSrvr::OnSellItem

```cpp
	if (nTax > 0)
		CTax::GetInstance()->AddTax(CTax::GetInstance()->GetContinent(pUser), nTax, TAX_SALES);


#ifdef __BUYBACK
	BuyBack::BuyBackPlayer& bbc = BuyBack::Sys.GetPlayer(pUser->m_idPlayer);
	if (bbc.IsFull())
	{
		bbc.RemoveAtIndex(0);
		bbc.Reorganize(); // todo: very inefficient.
		pUser->BuyBackRemoveItem(0);
	}
	pItemElem->m_nCost = nTotalPrice;

#ifdef __fixes // todo : missing CItemElem(const CITemeElem&) constructor copying raw pointer data
	CItemElem itemElem;
	itemElem = *pItemElem;
#else 
	CItemElem itemElem = *pItemElem;
#endif
	itemElem.m_nItemNum = nNum;
	pUser->SendBuyBack(&itemElem);
	bbc.Add(std::move(itemElem));
#endif 
	pUser->RemoveItem((BYTE)dwId, nNum);
```

```cpp
#ifdef __BUYBACK
void CDPSrvr::OnBuyBack(CAr& ar, DPID dpidCache, DPID dpidUser, LPBYTE lpBuf, u_long uBufSize)
{
	if (CUser* pUser = g_UserMng.GetUser(dpidCache, dpidUser); IsValidObj(pUser))
	{
		int nItem;
		ar >> nItem;

		BuyBack::BuyBackPlayer& Player_Ref = BuyBack::Sys.GetPlayer(pUser->m_idPlayer);
		if (CItemElem* pItemElem = Player_Ref.GetAtIndex(nItem))
		{
			if (const int nNum = pItemElem->m_nItemNum; nNum <= 0) {
				pUser->AddDefinedText(TID_GAME_LACKMONEY, "");
				return;
			}

			const __int64 nTotal = static_cast<__int64>(pItemElem->m_nCost());
			if (nTotal < 0 || nTotal > INT_MAX)
				return;

			if (pUser->GetGold() < nTotal)
			{
				pUser->AddDefinedText(TID_GAME_LACKMONEY);;
				return;
			}

			if (pUser->CreateItem(pItemElem))
			{
				pUser->AddGold(-nTotal);
				Player_Ref.RemoveAtIndex(nItem);
				Player_Ref.Reorganize();
				pUser->BuyBackRemoveItem(nItem);
			}
			else
				pUser->AddDefinedText(TID_GAME_LACKSPACE);
		}
	}
}
#endif
```

- dpsrvr.h
```cpp
#ifdef __BUYBACK
	void OnBuyBack(CAr& ar, DPID dpidCache, DPID dpidUser, LPBYTE lpBuf, u_long uBufSize);
#endif
```

- ThreadMng.cpp
```cpp
#ifdef __BUYBACK
#include "Buyback.h"
#endif

```

CRunObject::Run
```cpp
	CTimeout	timeoutCallTheRoll(MIN(1), MIN(1)); // This can be timed out for a minute on start anyway.
```
```cpp
			if (timeoutCallTheRoll.TimeoutReset(g_tmCurrent))
			{
				CCreateMonster::GetInstance()->ProcessRemoveMonster();
#ifdef __BUYBACK
				BuyBack::Sys.Process();
#endif

			}

```

- User.cpp
<small>*DestroyPlayer*</small>

```cpp
#ifdef __BUYBACK
	BuyBack::Sys.DisconnectUser(pUser->m_idPlayer);
#endif

	pUser->ResetCheckClientReq();
```

```cpp
#ifdef __BUYBACK
void CUser::SendEntireBuyBack(BuyBack::BuyBackPlayer& Player_Ref)
{
	if (IsDelete())
		return;
#ifdef __LatencyChanges_Test
	Queued_LowPrio_Snapshot.AddSnapshot(GetId(), SNAPSHOTTYPE_BUYBACK_ALL);
	Player_Ref.Serialize(Queued_LowPrio_Snapshot.ar);
#else
	m_Snapshot.AddSnapshot(GetId(), SNAPSHOTTYPE_BUYBACK_ALL);
	Player_Ref.Serialize(m_Snapshot.ar);
#endif
}


void CUser::SendBuyBack(CItemElem* pItemElem)
{
	if (IsDelete())	
		return;

	m_Snapshot.cb++;
	m_Snapshot.ar << GetId();
	m_Snapshot.ar << SNAPSHOTTYPE_BUYBACK;
	pItemElem->Serialize(m_Snapshot.ar);
}

void CUser::BuyBackRemoveItem(int nItem)
{
	if (IsDelete())	return;

	m_Snapshot.cb++;
	m_Snapshot.ar << GetId();
	m_Snapshot.ar << SNAPSHOTTYPE_BUYBACK_REMOVE_ITEM;
	m_Snapshot.ar << nItem;
}
#endif
```

- User.h (CUser)
```cpp
#ifdef __BUYBACK
	void SendEntireBuyBack(BuyBack::BuyBackPlayer& Player_Ref);
	void SendBuyBack(CItemElem* pItemElem);
	void BuyBackRemoveItem(int nItem);
#endif

```

- Item.h
```cpp
	virtual ~CItemContainer();

	void Clear();
#ifdef __BUYBACK 	
	[[nodiscard]] bool ContainsElements() {	 return GetCount() != 0; }
#endif
```

- WndControl.cpp
```cpp
#ifdef __BUYBACK
#include "BuyBack.h"
#endif
```
CWndListBox::OnDraw
```cpp
#ifdef __BUYBACK
	else if (pWnd->GetWndId() == APP_BUYBACK)
	{
		m_nFontHeight = 35;
		CPoint pt(10, 3);
		DWORD dwColor;
		dwColor = 0xFF000000;
		CRect rectClient = GetClientRect();
		int nPage = rectClient.Height() / m_nFontHeight;

		BuyBack::BuyBackPlayer& bbc = BuyBack::Sys.GetPlayer();
		const auto count = bbc.GetCount();
		m_wndScrollBar.SetScrollRange(0, count);
		m_wndScrollBar.SetScrollPage(nPage);
		for (unsigned i = static_cast<unsigned>(m_wndScrollBar.GetScrollPos()); i < count; ++i)
		{
			if (i > static_cast<unsigned>(nPage + m_wndScrollBar.GetScrollPos()))
				break;

			CItemElem* pItemElem = bbc.GetAtIndex(i);
			if (pItemElem)
			{
				p2DRender->RenderLine(CPoint(-2, pt.y + 35), CPoint(rectClient.right - 6, pt.y + 35), 0xFFB5BEBE);
				if (i == m_nCurSelect)
				{
					CRect DrawRect = CRect(0, pt.y + 2, rectClient.right - 6, pt.y + 34);
					p2DRender->RenderFillRect(DrawRect, 0xFFEECBBA);
					p2DRender->RenderRect(DrawRect, 0xFFCC6633);
				}
				const auto* prop = pItemElem->GetProp();
				if (CString strIcon = prop->szIcon; !strIcon.IsEmpty())
				{
					if (prop->dwItemKind3 == IK3_EGG)
					{
						unsigned char nLevel = 0;
						if (pItemElem->m_pPet)
							nLevel = pItemElem->m_pPet->GetLevel();
						switch (nLevel)
						{
							case PL_D:
							case PL_C:
								strIcon.Replace(".", "_00.");
								break;
							case PL_B:
							case PL_A:
								strIcon.Replace(".", "_01.");
								break;
							case PL_S:
								strIcon.Replace(".", "_02.");
								break;
							default:
								break;
						}
					}
					CTexture* pIcon = CWndBase::m_textureMng.AddTexture(g_Neuz.m_pd3dDevice, MakePath(DIR_ITEM, strIcon), 0xffff00ff);
					pIcon->Render(p2DRender, pt);
				}
				CRect rectToolTip(pt.x, pt.y, pt.x + 35, pt.y + 35);
				if (rectToolTip.PtInRect(m_ptMouse))
				{
					CPoint pt2 = m_ptMouse;
					ClientToScreen(&pt2);
					ClientToScreen(&rectToolTip);
					g_WndMng.PutToolTip_Item(&(*pItemElem), pt2, &rectToolTip, APP_BUYBACK);
				}
				pt.x += 40;
				pt.y += 10;

				CString szName = pItemElem->GetProp()->szName, szItemQt, szPrice;
				szItemQt.Format("%d", pItemElem->m_nItemNum);
				szPrice.Format("%I64d", static_cast<__int64>(pItemElem->m_nCost));
				szItemQt = GetNumberFormatEx(szItemQt);
				szPrice = GetNumberFormatEx(szPrice);

				if (szName.GetLength() > 24)
				{
					int	nReduceCount = 0;
					while (nReduceCount < 24)
					{
						if (IsDBCSLeadByte(szName[nReduceCount]))
							nReduceCount += 2;
						else
							nReduceCount++;
					}
					szName = szName.Left(nReduceCount);
					szName += "...";

					CRect rectToolTipName(pt.x - 10, pt.y, pt.x + 160, pt.y + 25);
					if (rectToolTipName.PtInRect(m_ptMouse))
					{
						CPoint pt2 = m_ptMouse;
						ClientToScreen(&pt2);
						ClientToScreen(&rectToolTipName);
						g_toolTip.PutToolTip(100, pItemElem->GetProp()->szName, rectToolTipName, pt2, 0);
					}
				}
				p2DRender->TextOut(pt.x, pt.y, szName, dwColor);
				p2DRender->TextOut(pt.x + 200 + ((3 - szItemQt.GetLength()) * 2), pt.y, szItemQt, dwColor);
				p2DRender->TextOut(pt.x + 280 + ((13 - szPrice.GetLength()) * 2), pt.y, szPrice, dwColor);

				pt.x = 10;
				pt.y += m_nFontHeight - 10;
			}
		}
	}
#endif
```

- WndShop.cpp
```cpp
CWndShop::CWndShop()
{
	m_pMover = NULL;
	m_pWndConfirmSell = NULL;
#ifdef __BUYBACK
	m_pWndBuyBack = NULL;
#endif
}

CWndShop::~CWndShop()
{
	SAFE_DELETE(m_pWndConfirmSell);
	SAFE_DELETE(g_WndMng.m_pWndTradeGold);
#ifdef __BUYBACK
	SAFE_DELETE(m_pWndBuyBack);
#endif
}
```

<small>CWndShop::OnChildNotify</small>

```cpp
						SAFE_DELETE(m_pWndConfirmSell);
#ifdef __BUYBACK
						SAFE_DELETE(m_pWndBuyBack);
#endif

						m_pWndConfirmSell = new CWndConfirmSell;
```
```cpp
#ifdef __BUYBACK
	case WIDC_BUTTON33:
	{
		SAFE_DELETE(m_pWndBuyBack);
		m_pWndBuyBack = new BuyBack::CWndBuyBack;
		m_pWndBuyBack->Initialize(this);
	}
	break;
#endif

	}

	return CWndNeuz::OnChildNotify(message, nID, pLResult);
```

- WndShop.h
```cpp
class CWndShop : public CWndNeuz
{
public:
#ifdef __BUYBACK
	BuyBack::CWndBuyBack* m_pWndBuyBack;
#endif
```

- WndWorld.cpp
<small>*CWndWorld::OnCommand*</small>

```cpp
#ifdef __BUYBACK
		case MMI_BUYBACK:
		{
			if (!g_WndMng.GetWndBase(APP_BUYBACK))
			{
				CWndBuyBack* pBuyBack = new CWndBuyBack();
				pBuyBack->Initialize();
			}
		}
		break;
#endif
```

<small>*CWndWorld::Process*</small>

```cpp
#ifdef __MODEL_CHANGE
			if (g_WndMng.m_pWndLookChange)
				g_WndMng.m_pWndLookChange->Destroy();
#endif
#ifdef __BUYBACK
			pWndBase = g_WndMng.GetWndBase(APP_BUYBACK);
			if (pWndBase)
				pWndBase->Destroy();
#endif 
```


- Buyback.h
```cpp
#pragma once

#include "item.h"
#include <map>
#ifdef __CLIENT
#include "WndNeuz.h"
#endif

#ifdef __BUYBACK
namespace BuyBack
{
	inline size_t Maximum = 50;
	inline time_t Max_DC_Timer = MIN(5);
	inline bool Allow_DC_Timer = true;

	constexpr unsigned long PACKETTYPE_REQUEST_BUYBACK = 0xAA000042;
	constexpr unsigned long PACKETTYPE_SEND_BUYBACK = 0xAA000041;
	constexpr unsigned short SNAPSHOTTYPE_BUYBACK = 0x8933;
	constexpr unsigned short SNAPSHOTTYPE_BUYBACK_REMOVE_ITEM = 0x8934;
	constexpr unsigned short SNAPSHOTTYPE_BUYBACK_ALL = 0x8935;

	class BuyBackPlayer
	{
		CItemContainer<CItemElem> Fake_Inventory;
		time_t DC_Timer = 0;
#ifdef __CLIENT
		bool CanBuy = true;
#endif

		public:
			BuyBackPlayer() {
				Fake_Inventory.SetItemContainer(ITYPE_ITEM, Maximum);
			}
			[[nodiscard]] time_t GetDCTimer() const noexcept {
				return DC_Timer;
			}
			void SetDCTimer(const time_t t) noexcept {
				DC_Timer = t;
			}
			[[nodiscard]] bool ContainsElements() { return Fake_Inventory.ContainsElements(); }
			bool Add(CItemElem&& item);
			bool RemoveAtIndex(unsigned char SlotID);
			void Reorganize();
			void Serialize(CAr &ar);
			CItemElem* GetAtIndex(unsigned char SlotID);
			bool IsFull();
			unsigned long GetCount() { return Fake_Inventory.GetCount(); }
	};

	class BuyBackManager
	{
#ifdef __WORLDSERVER
		std::map<unsigned long, BuyBackPlayer> Player_List;
#else
		BuyBackPlayer Player_List;
#endif

		public:
		static void Load();

			BuyBackPlayer& GetPlayer(
#ifdef __WORLDSERVER
				unsigned long Player_ID
#endif
			);
#ifdef __WORLDSERVER
			void Process();
			void DisconnectUser(unsigned long Player_ID);
#endif

			bool Add(CItemElem&& item
#ifdef __WORLDSERVER
				, unsigned long Player_ID
#endif			
			);

			bool RemoveAtIndex(unsigned char SlotID
#ifdef __WORLDSERVER
				, unsigned long Player_ID
#endif
			);

			void Reorganize(
#ifdef __WORLDSERVER
				unsigned long Player_ID
#endif
			);
#ifdef __CLIENT
			void Serialize(CAr& ar);
#endif

	};
	inline BuyBackManager Sys;


#ifdef __CLIENT

	class CWndBuyBack final : public CWndNeuz
	{
		public:
			CWndBuyBack();
			~CWndBuyBack() override;

			void OnDraw(C2DRender* p2DRender) override;
			void OnInitialUpdate() override;
			BOOL Initialize(CWndBase* pWndParent = NULL, DWORD nType = MB_OK) override;
			BOOL OnCommand(UINT nID, DWORD dwMessage, CWndBase* pWndBase) override;
			void OnSize(UINT nType, int cx, int cy) override;
			BOOL OnChildNotify(UINT message, UINT nID, LRESULT* pLResult) override;
			virtual void ProcessListBox();
	};
#endif

}

#endif
```



- Buyback.cpp
```cpp
#include "StdAfx.h"
#include "BuyBack.h"

#ifdef __CLIENT
#include "Resdata.h"
#include "dpclient.h"
extern CDPClient g_DPlay;
#endif

#ifdef __BUYBACK
namespace BuyBack
{
	bool BuyBackPlayer::Add(CItemElem&& item)
	{
		return Fake_Inventory.Add(&item);
	}

	bool BuyBackPlayer::RemoveAtIndex(const unsigned char SlotID)
	{
		Fake_Inventory.RemoveAtId(SlotID);
		return true;
	}

	void BuyBackPlayer::Serialize(CAr& ar)
	{
		Fake_Inventory.Serialize(ar);
	}

	void BuyBackPlayer::Reorganize()
	{
		CItemContainer<CItemElem> tmp;
		tmp.SetItemContainer(ITYPE_ITEM, Fake_Inventory.GetMax());
		CItemElem* pItemElem;
		for (int i = 0; i < Fake_Inventory.GetMax(); i++)
		{
			pItemElem = Fake_Inventory.GetAt(i);
			if (pItemElem)
			{
				tmp.Add(pItemElem);
			}
		}

		Fake_Inventory.Clear();
		for (int i = 0; i < tmp.GetMax(); i++)
		{
			pItemElem = tmp.GetAt(i);
			if (pItemElem)
			{
				Fake_Inventory.Add(pItemElem);
			}
		}
	}
	CItemElem* BuyBackPlayer::GetAtIndex(const unsigned char SlotID)
	{
		return Fake_Inventory.GetAt(SlotID);
	}
	[[nodiscard]] bool BuyBackPlayer::IsFull()
	{
		return Fake_Inventory.GetCount() >= Fake_Inventory.GetMax();
	}


	void BuyBackManager::Load()
	{
		CScript s;
		if (!s.Load("BuyBackInfo.txt"))
			return;

		s.GetToken();
		while (s.tok != FINISHED)
		{
			if (s.Token == "Maximum")
				Maximum = s.GetNumber();
			else if (s.Token == "Max_DC_Timer")
				Max_DC_Timer = s.GetNumber();
			else if (s.Token == "Allowe_DC_Timer")
				Allow_DC_Timer = s.GetNumber() != 0;
			s.GetToken();
		}
	}

	BuyBackPlayer& BuyBackManager::GetPlayer(
#ifdef __WORLDSERVER
	const unsigned long Player_ID
#endif
	) 
	{
#ifdef __WORLDSERVER
			auto& res = Player_List[Player_ID];
			if (res.GetDCTimer() != 0)
				res.SetDCTimer(0);
				
			return res;
#else
			return Player_List;
#endif
	}

#ifdef __WORLDSERVER
	void BuyBackManager::Process()
	{
		for (auto iter = Player_List.begin(); iter != Player_List.end(); )
		{
			if (const auto dctime = iter->second.GetDCTimer(); dctime != 0 && dctime > g_tmCurrent)
				iter = Player_List.erase(iter);
			else
				++iter;
		}
	}
	void BuyBackManager::DisconnectUser(const unsigned long Player_ID)
	{
		if (!Allow_DC_Timer)
		{
			if (const auto location = Player_List.find(Player_ID); location != Player_List.end())
				Player_List.erase(location);
		}
		else
			Player_List[Player_ID].SetDCTimer(g_tmCurrent);
	}
#endif


	bool BuyBackManager::Add(CItemElem&& item
#ifdef __WORLDSERVER
		, const unsigned long Player_ID
#endif
	)
	{
#ifdef __CLIENT
		Player_List.Add(std::move(item));
#else
		if (const auto location = Player_List.find(Player_ID); location != Player_List.end())
			location->second.Add(std::move(item));
#endif

		return true;
	}

	bool BuyBackManager::RemoveAtIndex(const unsigned char SlotID
#ifdef __WORLDSERVER
		, const unsigned long Player_ID
#endif
	)
	{
#ifdef __CLIENT
		return Player_List.RemoveAtIndex(SlotID);
#else
		if (const auto location = Player_List.find(Player_ID); location != Player_List.end())
		{
			location->second.RemoveAtIndex(SlotID);
			return true;
		}
		return false;
#endif
	}

	void BuyBackManager::Reorganize(
#ifdef __WORLDSERVER
		const unsigned long Player_ID
#endif
	)
	{
#ifdef __WORLDSERVER
		if (const auto location = Player_List.find(Player_ID); location != Player_List.end())
			location->second.Reorganize();

#else
		Player_List.Reorganize();
#endif
	}
#ifdef __CLIENT
	void BuyBackManager::Serialize(CAr& ar)
	{
		Player_List.Serialize(ar);
	}
#endif




#ifdef __CLIENT
	CWndBuyBack::CWndBuyBack() = default;
	CWndBuyBack::~CWndBuyBack() = default;

	void CWndBuyBack::OnDraw(C2DRender* p2DRender)
	{
		CWndButton* pWndOk = static_cast<CWndButton*>(GetDlgItem(WIDC_OK));
		pWndOk->EnableWindow(static_cast<CWndListBox*>(GetDlgItem(WIDC_LISTBOX1))->GetCurSel() != -1);
	}

	void CWndBuyBack::OnInitialUpdate()
	{
		CWndNeuz::OnInitialUpdate();
		ProcessListBox();
		MoveParentCenter();
	}

	BOOL CWndBuyBack::Initialize(CWndBase* pWndParent, DWORD)
	{
		return InitDialog(g_Neuz.GetSafeHwnd(), APP_BUYBACK, 0, CPoint(0, 0), pWndParent);
	}

	BOOL CWndBuyBack::OnCommand(UINT nID, DWORD dwMessage, CWndBase* pWndBase)
	{
		return CWndNeuz::OnCommand(nID, dwMessage, pWndBase);
	}

	void CWndBuyBack::OnSize(UINT nType, int cx, int cy) 
	{
		CWndNeuz::OnSize(nType, cx, cy);
	}

	BOOL CWndBuyBack::OnChildNotify(UINT message, UINT nID, LRESULT* pLResult)
	{
		int nCurSel = static_cast<CWndListBox*>(GetDlgItem(WIDC_LISTBOX1))->GetCurSel();
		switch (nID)
		{
			case WIDC_OK:
				if (nCurSel != -1)
					g_DPlay.SendBuyBack(nCurSel);
				break;
			case WIDC_CANCEL:
				Destroy();
				break;
			default:
				break;
		}
		return CWndNeuz::OnChildNotify(message, nID, pLResult);
	}

	void CWndBuyBack::ProcessListBox()
	{
		if (CWndListBox* pListBox = static_cast<CWndListBox*>(GetDlgItem(WIDC_LISTBOX1)))
		{
			pListBox->ResetContent();
			auto& Player = Sys.GetPlayer();
			for (int i = 0; i < Player.GetCount(); i++)
				pListBox->AddString("");
		}
	}
#endif
}
#endif
```

> Note: There might be some issues. It's been a while and this is also untested.



## License
---
_FlyFF is an mmorpg created by GalaNet/Aeonsoft. This implementation was done by myself:_
| Account  | Site |
| -------- | ------- |
| Kiaemi. | Discord |
| Velvet | RageZone |
| M¿dScientist | Elitepvpers |
