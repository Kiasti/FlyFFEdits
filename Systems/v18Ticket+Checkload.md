# New Ticket
---

[![Smelt Saftey](http://img.youtube.com/vi/fl9AYsQS_G0/0.jpg)](https://www.youtube.com/watch?v=fl9AYsQS_G0)


## Features
---
- V18 ticket system.
- Multi-World tickets.
- With this system, you'll be able to "checkload" which will get and return player positions from each map channel. You're able to swap channels, etc.

> Note: There might be things missing, or compile errors. This was grabbed from an old source and I started to alter lower camelCase to upper CamelCase.

<br>

### V18 Ticket
---
- **DpClient.cpp** *CDPClient::SendDoUseItem* 
	<sub>*In a florist base, this would be in: CWndMgr::UseItem_MaterialWnd2*</sub>
    ```CPP
		if( !pItem->IsFlag( CItemElem::expired ) )
		{
	#ifdef __V18Ticket
			CWorld* pWorld = g_pPlayer->GetWorld();
			auto worldId = pWorld->GetID();
			if (worldId != WI_WORLD_GUILDWAR || !(worldId >= WI_WORLD_GUILDWAR1TO1_0 && worldId <= 	WI_WORLD_GUILDWAR1TO1_L))
			// Add your pvp maps to stop tickets from opening in pvp maps.
			// You can ideally check if the user is in the state of pvp or osmethin.
			{
				if (pItemProp->dwItemKind3 == IK3_TICKET)
				{
					if (g_WndMng.m_pWndSelectCh)
					{
						delete g_WndMng.m_pWndSelectCh;
						g_WndMng.m_pWndSelectCh = nullptr;
					}
					g_WndMng.m_pWndSelectCh = new CWndSelectCh(dwItemId,  pItemProp->dwID);
					g_WndMng.m_pWndSelectCh->Initialize(&g_WndMng);
					return;
				}
			}
	#else
			TicketProp* pTicketProp	= CTicketProperty::GetInstance()->GetTicketProp( pItemProp->dwID );
			if( pTicketProp && g_pPlayer )
			{
				if( g_pPlayer->GetWorld()->GetID() == pTicketProp->dwWorldId )
				{
					SAFE_DELETE( g_WndMng.m_pWndCommItemDlg );
					g_WndMng.m_pWndCommItemDlg	= new CWndCommItemDlg;
					g_WndMng.m_pWndCommItemDlg->Initialize( &g_WndMng, APP_COMMITEM_DIALOG );
					g_WndMng.m_pWndCommItemDlg->SetItem( TID_GAME_TICKET_DESC, dwId,  pItemProp->dwID );
					return;
				}
				else
				{
					// È®Àå ·¹ÀÌ¾î °³¼ö
					int nExpand	= CTicketProperty::GetInstance()	->GetExpanedLayer( pTicketProp->dwWorldId );
					SAFE_DELETE( g_WndMng.m_pWndSelectCh );
					g_WndMng.m_pWndSelectCh = new CWndSelectCh(dwItemId, nExpand);
					g_WndMng.m_pWndSelectCh->Initialize(&g_WndMng);
				}
			}
	#endif
    ```

- **Item.cpp** *CItemElem::IsActiveTicket*
	```cpp
	#ifdef __V18Ticket
			if (!CTicketProperty::GetInstance()->IsMultiWorldTicket	(m_dwItemId))
			{
				const TicketProp* pProp1 = CTicketProperty::GetInstance()->GetTicketProp(m_dwItemId);
				const TicketProp* pProp2 = CTicketProperty::GetInstance()->GetTicketProp(dwItemId);
				return pProp1->dwWorldId == pProp2->dwWorldId;
			}
			if (CTicketProperty::GetInstance()->IsMultiWorldTicket(dwItemId))
				return true;
	#else
			TicketProp* pProp1 = CTicketProperty::GetInstance()->GetTicketProp(m_dwItemId);
			TicketProp* pProp2 = CTicketProperty::GetInstance()->GetTicketProp(dwItemId);
			return (pProp1->dwWorldId == pProp2->dwWorldId);
	#endif
	```

- **Ticket.h**
	```cpp
	#pragma once

	typedef	struct _TicketProp
	{
		DWORD dwWorldId;
		D3DXVECTOR3	vPos;
	} TicketProp, *PTicketProp;

	// This code that contains "layer" information can be intergrated fully
	// with the ticket system.
	#ifndef __V18Ticket
	typedef struct	_LayerStruct
	{
		DWORD dwWorldId;
		int nExpand;
	}	LayerStruct;


	class CLayerProperty
	{
		std::vector<LayerStruct> m_vLayers;
	public:
		CLayerProperty();
		virtual ~CLayerProperty();
		bool LoadScript();
		int	GetExpanedLayer(const unsigned long dwWorldId) const;
	};
	#endif


	class CTicketProperty
	{
	#ifdef __V18Ticket
		std::multimap<unsigned long, unsigned long> itemToIndex;
		std::map<unsigned long, TicketProp> indexToTicketProp;
		std::map<unsigned long, int> expandedLayers;
	#else
		std::map<DWORD, TicketProp>	m_mapTicketProp;
		CLayerProperty m_lp;
	#endif

	public:
		CTicketProperty();
		virtual	~CTicketProperty();
		static CTicketProperty* GetInstance();

	#ifdef __V18Ticket
		bool isMultiWorldTicket(const unsigned long itemId) const;
		std::vector<const TicketProp*> getMultiworldTicketProp(const unsigned long itemId) const;
		bool isInTicketedWorld(const unsigned long dwWorldId) const;
		bool LoadScript();
		const TicketProp* getTicketProp(const unsigned long dwItemId) const;
		int getExpandedLayer(const unsigned long dwWorldId) const;
	#else
		TicketProp*	GetTicketProp(DWORD dwItemId);
		BOOL IsTarget(DWORD dwWorldId);
		BOOL LoadScript();
		int GetExpanedLayer(const unsigned long dwWorldId) const { return m_lp.GetExpanedLayer(dwWorldId); }
	#endif
	};
	```

- **Ticket.cpp** 
	```cpp
	#include "stdafx.h"
	#include "ticket.h"

	CTicketProperty::CTicketProperty() = default;
	CTicketProperty::~CTicketProperty() = default;


	#ifdef __V18Ticket
	bool CTicketProperty::IsInTicketedWorld(const unsigned long dwWorldId) const
	{
		for (auto it = indexToTicketProp.cbegin(); it != indexToTicketProp.cend(); ++it)
		{
			if (dwWorldId == it->second.dwWorldId)
				return true;
		}
		return false;
	}
	bool CTicketProperty::IsMultiWorldTicket(const unsigned long itemId) const
	{
		return itemToIndex.count(itemId) > 1;
	}

	std::vector<const TicketProp*> CTicketProperty::GetMultiworldTicketProp(const unsigned long itemId) const
	{
		std::vector<const TicketProp*> tempVector;
		const auto result = itemToIndex.equal_range(itemId);
		for (auto it = result.first; it != result.second; ++it)
		{
			auto it2 = indexToTicketProp.find(it->second);
			tempVector.push_back(it2 != indexToTicketProp.end() ? &it2->second : nullptr);
		}
		return tempVector;
	}

	const TicketProp* CTicketProperty::GetTicketProp(const unsigned long dwItemId) const
	{
		if (const auto it = itemToIndex.find(dwItemId); it != itemToIndex.end())
		{
			if (const auto it2 = indexToTicketProp.find(it->second); it2 != indexToTicketProp.end())
				return &it2->second;
		}
		return nullptr;
	}

	int CTicketProperty::GetExpandedLayer(const unsigned long dwWorldId) const
	{
		if (const auto it = expandedLayers.find(dwWorldId); it != expandedLayers.end())
			return it->second;
		return 0;
	}

	#else
	BOOL	CTicketProperty::IsTarget( DWORD dwWorldId )
	{
	for(std::map<DWORD, TicketProp>::iterator i = m_mapTicketProp.begin(); i != m_mapTicketProp.end(); ++i )
		{
			if( i->second.dwWorldId == dwWorldId )
				return TRUE;
		}
		return FALSE;
	}

	TicketProp*	CTicketProperty::GetTicketProp( DWORD dwItemId )
	{
		std::map<DWORD, DWORD>::iterator it = m_mapTicketLink.find(dwItemId);
		if (it == m_mapTicketLink.end())
			return NULL;
		std::map<DWORD, TicketProp>::iterator i = m_mapTicketProp.find(it->second);
		if (i != m_mapTicketProp.end())
			return &i->second;
		return NULL;
	}
	#endif


	CTicketProperty* CTicketProperty::GetInstance()
	{
		static	CTicketProperty sTicketProperty;
		return &sTicketProperty;
	}

	BOOL CTicketProperty::LoadScript()
	{
		CScript s;

	#ifdef __V18Ticket
		if (s.Load("PaidWorldSet.txt") == FALSE)
			return FALSE;

		DWORD dwIndex = s.GetNumber();
		while (s.tok != FINISHED)
		{
			TicketProp tp;
			tp.dwWorldId = s.GetNumber();
			tp.vPos.x = s.GetFloat();
			tp.vPos.y = s.GetFloat();
			tp.vPos.z = s.GetFloat();
			s.GetToken();
			indexToTicketProp.insert(std::make_pair(dwIndex, tp));
			dwIndex = s.GetNumber();
		}

		if (s.Load("PaidWorldTicket.txt") == FALSE)
			return FALSE;
		dwIndex = s.GetNumber();
		DWORD dwWorldId = 0;
		while (s.tok != FINISHED)
		{
			dwWorldId = s.GetNumber();
			itemToIndex.insert(std::make_pair(dwIndex, dwWorldId));
			dwIndex = s.GetNumber();
		}
		if (!s.Load("layer.inc"))
			return false;

		dwWorldId = s.GetNumber();
		while (s.tok != FINISHED)
		{
			const int nExpand = s.GetNumber();
			expandedLayers.insert(std::make_pair(dwWorldId, nExpand));
			dwWorldId = s.GetNumber();
		}

	#else

		if (s.Load("ticket.inc") == FALSE)
			return FALSE;

		DWORD dwItemId = s.GetNumber();
		while (s.tok != FINISHED)
		{
			TicketProp	tp;
			tp.dwWorldId = s.GetNumber();
			tp.vPos.x = s.GetFloat();
			tp.vPos.y = s.GetFloat();
			tp.vPos.z = s.GetFloat();
			bool b = m_mapTicketProp.insert(std::map<DWORD, TicketProp>::value_type(dwItemId, tp)).second;
			ASSERT(b);
			dwItemId = s.GetNumber();
		}
		return m_lp.LoadScript();

	#endif
		return true;
	}

	#ifndef __V18Ticket
	#ifdef __AZRIA_1023
	CLayerProperty::CLayerProperty()
	{
	}

	CLayerProperty::~CLayerProperty()
	{
	}

	BOOL CLayerProperty::LoadScript()
	{
		CScript s;
		if( s.Load( "layer.inc" ) == FALSE )
			return FALSE;
		DWORD dwWorldId	= s.GetNumber();
		while( s.tok != FINISHED )
		{
			LayerStruct ls;
			ls.dwWorldId	= dwWorldId;
			ls.nExpand	= s.GetNumber();
			m_vLayers.push_back( ls );
			dwWorldId	= s.GetNumber();
		}
		return TRUE;
	}

	int CLayerProperty::GetExpanedLayer( DWORD dwWorldId )
	{
		for( VLS::iterator i = m_vLayers.begin(); i != m_vLayers.end(); ++i )
			if( ( *i ).dwWorldId == dwWorldId )
				return ( *i ).nExpand;
		return 0;
	}
	#endif	// __AZRIA_1023
	#endif
	```

- **WndSelectCh.h**
	```cpp
	#ifndef __WNDSELECTCH__H
	#define __WNDSELECTCH__H

	#ifdef __V18Ticket
	#include "ticket.h"
	#endif

	class CWndSelectCh final : public CWndNeuz
	{
	#ifdef __V18Ticket
		std::vector<const TicketProp*> worldList;
		CWndListBox* pWndListBox;
		CWndListBox* pWndListBox2;
		unsigned long propItemId;
		unsigned long mixedVar;

		void CreateWorldListBox() const;
		void CreateChannelListBox(unsigned long index = 0) const;
		bool CheckWorld();
	#else
		int m_nItemId;
		int m_nChCount;
	#endif

		public:
	#ifdef __V18Ticket
			CWndSelectCh(unsigned long mixVar, unsigned long propId);
	#else
			CWndSelectCh(int nItemId, int nChCount);
	#endif
			~CWndSelectCh() override;

			BOOL Initialize(CWndBase* pWndParent = NULL, DWORD nType = MB_OK) override;
			BOOL OnChildNotify(UINT message, UINT nID, LRESULT* pLResult) override;
			void OnDraw(C2DRender* p2DRender) override;
			void OnInitialUpdate() override;
			BOOL OnCommand(UINT nID, DWORD dwMessage, CWndBase* pWndBase) override;
			void OnSize(UINT nType, int cx, int cy) override;
			void OnLButtonUp(UINT nFlags, CPoint point) override;
			void OnLButtonDown(UINT nFlags, CPoint point) override;
	};
	#endif
	```

- **WndSelectCh.cpp**
	```cpp
	#include "stdafx.h"
	#include "defineText.h"
	#include "AppDefine.h"

	#include "WndSelectCh.h"
	#include "DPClient.h"
	extern	CDPClient	g_DPlay;


	#ifdef __V18Ticket
	CWndSelectCh::CWndSelectCh(const unsigned long mixVar, const unsigned long propId) : propItemId(propId), mixedVar(mixVar)
	{
	}

	void CWndSelectCh::CreateWorldListBox() const
	{
		for (auto it = worldList.cbegin(); it != worldList.cend(); ++it)
		{
			if (!*it) 
				continue;

			const auto* world_data = g_WorldMng.m_aWorld.GetAt((*it)->dwWorldId);
			if (!world_data)
				continue;

			const int nIndex = pWndListBox2->AddString(world_data->m_szWorldName);
			unsigned long* newData = new unsigned long;
			*newData = (*it)->dwWorldId;
			pWndListBox2->SetItemDataPtr(nIndex, newData);
		}
	}
	void CWndSelectCh::CreateChannelListBox(const unsigned long index) const
	{
		pWndListBox->ResetContent();
		if (const unsigned long* worldId = static_cast<unsigned long*>(pWndListBox2->GetItemDataPtr(static_cast<int>(index))))
		{
			const int expand = CTicketProperty::GetInstance()->GetExpandedLayer(*worldId) + 1;
			std::string tmp = prj.GetText(TID_GAME_CHAR_SERVERNAME);
			tmp += " ";

			for (int i = 0; i < expand; ++i)
			{
				std::string loopStr = tmp + std::to_string(i + 1);
				pWndListBox->AddString(loopStr.c_str());
			}
		}
	}

	bool CWndSelectCh::CheckWorld()
	{
	#ifndef __ChannelSwitcher
		for (std::vector<const TicketProp*>::const_iterator it = worldList.begin(); it != worldList.end(); ++it)
		{
			if (g_pPlayer->GetWorld()->GetID() == (*it)->dwWorldId)
			{
				pWndListBox2->ResetContent();
				pWndListBox->ResetContent();

				Destroy();

				SAFE_DELETE(g_WndMng.m_pWndCommItemDlg);
				g_WndMng.m_pWndCommItemDlg = new CWndCommItemDlg;
				g_WndMng.m_pWndCommItemDlg->Initialize(&g_WndMng, APP_COMMITEM_DIALOG);
				g_WndMng.m_pWndCommItemDlg->SetItem(TID_GAME_TICKET_DESC, HIWORD(mixedVar), propItemId);
				return true;
			}
		}
	#endif
		return false;
	}

	#else
	CWndSelectCh::CWndSelectCh(int nItemId, int nChCount)
	{
		m_nItemId  = nItemId;
		m_nChCount = nChCount + 1;
	}
	#endif

	CWndSelectCh::~CWndSelectCh() = default;

	void CWndSelectCh::OnDraw( C2DRender* p2DRender )
	{
	}


	void CWndSelectCh::OnInitialUpdate()
	{
		CWndNeuz::OnInitialUpdate();

	#ifdef __V18Ticket
		pWndListBox = dynamic_cast<CWndListBox*>(GetDlgItem(WIDC_LB_CHANNEL));
		pWndListBox2 = dynamic_cast<CWndListBox*>(GetDlgItem(WIDC_LB_WORLD));

		if (const CTicketProperty* tickInst = CTicketProperty::GetInstance(); tickInst->IsMultiWorldTicket(propItemId))
			worldList = tickInst->GetMultiworldTicketProp(propItemId);
		else
		{
			const TicketProp* tmpTicket = tickInst->GetTicketProp(propItemId);
			worldList.push_back(tmpTicket);
		}
		if (CheckWorld())
			return;
		CreateWorldListBox();
		CreateChannelListBox();

	#else
		CWndListBox* pWndListBox = (CWndListBox*)GetDlgItem( WIDC_LISTBOX1 );
		pWndListBox->ResetContent();
		CString strTitle;
		for(int i=0; i < m_nChCount; ++i)
		{
			strTitle.Format( "%s	%d", prj.GetText(TID_GAME_CHAR_SERVERNAME), i+1);
			pWndListBox->AddString(strTitle);
		}
	#endif

		pWndListBox->SetCurSel(0);
		pWndListBox2->SetCurSel(0);
		MoveParentCenter();
	}

	BOOL CWndSelectCh::Initialize( CWndBase* pWndParent, DWORD  )
	{
	#ifdef __V18Ticket
		return InitDialog(g_Neuz.GetSafeHwnd(), APP_WORLD_FREETICKET, 0, CPoint(0, 0), pWndParent);
	#else
		return CWndNeuz::InitDialog(g_Neuz.GetSafeHwnd(), APP_SELECT_CHANNEL, 0, CPoint(0, 0), pWndParent);
	#endif

	}

	BOOL CWndSelectCh::OnCommand( UINT nID, DWORD dwMessage, CWndBase* pWndBase )
	{
		return CWndNeuz::OnCommand( nID, dwMessage, pWndBase );
	}
	void CWndSelectCh::OnSize( UINT nType, int cx, int cy )
	{
		CWndNeuz::OnSize( nType, cx, cy );
	}
	void CWndSelectCh::OnLButtonUp( UINT nFlags, CPoint point )
	{
	}
	void CWndSelectCh::OnLButtonDown( UINT nFlags, CPoint point )
	{
	}
	BOOL CWndSelectCh::OnChildNotify( UINT message, UINT nID, LRESULT* pLResult )
	{
		switch(nID)
		{
	#ifdef __V18Ticket
			case WIDC_LB_WORLD:
				if (pWndListBox2->GetCurSel() >= 0)
					CreateChannelListBox(pWndListBox2->GetCurSel());
				pWndListBox->SetCurSel(0);
				break;
			case WIDC_LB_CHANNEL:
				if (message == WNM_DBLCLK)
				{

				}
				break;
			case WIDC_BT_MOVE:
			{
				unsigned long sendVar = pWndListBox->GetCurSel();
				if (static_cast<long>(sendVar) == -1)
					break;

				if (pWndListBox2->GetCurSel() >= 0)
				{
					if (const unsigned long* worldId = static_cast<unsigned long*>(pWndListBox2->GetItemDataPtr(pWndListBox2->GetCurSel())))
						sendVar = *worldId << 16 | sendVar;
				}

				const std::string var = std::to_string(sendVar);
				g_DPlay.SendDoUseItem(mixedVar, var.c_str());
				pWndListBox2->ResetContent();
				pWndListBox->ResetContent();
				Destroy();
				break;
			}
			case WIDC_BUTTON2: //check load future
	#ifdef __CheckLoad
				if (pWndListBox2->GetCurSel() >= 0)
				{
					if (const int layer = pWndListBox->GetCurSel(); layer != -1)
					{
						if (unsigned long* worldId = static_cast<unsigned long*>(pWndListBox2->GetItemDataPtr(pWndListBox2->GetCurSel())))
							g_DPlay.GetCheckLoad(*worldId, layer * -1);
					}
				}
				break;
	#endif
	#ifdef __ChannelSwitcher
			case WIDC_BUTTON11:
				g_DPlay.SendDoUseItem(mixedVar, std::to_string(1 << 16).c_str());
				pWndListBox2->ResetContent();
				pWndListBox->ResetContent();
				Destroy();
				break;
	#endif

	#else
			case WIDC_LISTBOX1: // view ctrl
				{
					CWndListBox* pWndListBox = (CWndListBox*)GetDlgItem( WIDC_LISTBOX1 );
					char strTemp[8];
					nSelect = pWndListBox->GetCurSel() * -1;
					_itoa(nSelect, strTemp, 10);
					g_DPlay.SendDoUseItem(m_nItemId, strTemp);
					Destroy();
				}
				break;
	#endif
			default:
				break;

		}

		return CWndNeuz::OnChildNotify( message, nID, pLResult );
	}
	```
- **DpSrvr.cpp**
	```cpp
	#ifdef __CheckLoad
		ON_MSG(PACKETTYPE_RequestCheckLoad, &CDPSrvr::OnCheckLoad);
	#endif
	```
	```cpp
	#ifdef __CheckLoad 
	#include "ticket.h"
	void CDPSrvr::OnCheckLoad(CAr& ar, DPID dpidCache, DPID dpidUser, LPBYTE lpBuf, u_long uBufSize)
	{
		const auto [worldId, layer] = ar.Extract<unsigned long, int>();

		if (CUser* pUser = g_UserMng.GetUser(dpidCache, dpidUser); IsValidObj(pUser))
		{
			auto& sussy_stuff = pUser->sus.GetAttempts(AttemptType::Timer_CheckLoad);
			if (sussy_stuff + SEC(2) > g_tmCurrent) {
				pUser->AddText("DontSpam");
				return;
			}
			sussy_stuff = g_tmCurrent + SEC(2);

			if (CTicketProperty::GetInstance()->IsInTicketedWorld(worldId))
			{
				if (CWorld* pWorld = g_WorldMng.GetWorld(worldId))
				{
					if (pWorld->m_linkMap.GetLinkMap(layer))
					{
						std::vector<D3DXVECTOR3> playerPositions;
						playerPositions.reserve(100);

						const D3DXVECTOR3 tmpVec{};
						CObj* pObj;
						FOR_LINKMAP(pWorld, tmpVec, pObj, 99999, CObj::linkPlayer, layer)
						{
							if (IsValidObj(pObj))
								playerPositions.emplace_back(pObj->m_vPos);
						}
						END_LINKMAPBREAK(playerPositions.size() >= 100);
						playerPositions.shrink_to_fit();

						pUser->SendCheckLoad(worldId, std::move(playerPositions));
					}
				}
			}
		}
	}
	#endif
	```

- **User.h**
	```cpp
	// I thought just using this to do time checking to stop spam.
	enum class AttemptType : unsigned char
	{
		DistanceIncorrect,
		MeleeAttack_IncorrectHitNum,
		Timer_CheckLoad,
		Max
	};

	class Sussy
	{
		std::unordered_map<AttemptType, unsigned long> Attempts;
		public:
			unsigned long& GetAttempts(AttemptType&& at) {
				return Attempts[at];
			}
	};
	```

	```cpp
	
	Sussy sus;
	#ifdef __CheckLoad
		void SendCheckLoad(unsigned long worldId, std::vector<D3DXVECTOR3>&& positions);
	#endif
	```
- User.cpp
	```cpp
	#ifdef __CheckLoad
	void CUser::SendCheckLoad(unsigned long worldId, std::vector<D3DXVECTOR3>&& playerPositions)
	{
		if (IsDelete())	
			return;

		m_Snapshot.cb++;
		m_Snapshot.ar << NULL_ID << SNAPSHOTTYPE_CheckLoad;
		m_Snapshot.ar << worldId << playerPositions;
	}
	```
	```cpp
	void CUser::DoUseItemTicket(CItemElem* pItemElem)
	{
		if (g_GuildCombat1to1Mng.IsPossibleUser(this))
			return;

		ItemProp* pItemProp = pItemElem->GetProp();
	#ifndef __ChannelSwitcher
		if (HasBuffByIk3(pItemProp->dwItemKind3))
		{
			if (!HasBuff(BUFF_ITEM, (WORD)(pItemElem->m_dwItemId)))
			{
				AddDefinedText(TID_GAME_MUST_STOP_OTHER_TICKET);
				return;
			}
			if (pItemElem->m_dwKeepTime == 0)
			{
				AddDefinedText(TID_GAME_LIMITED_USE);
				return;
			}
			RemoveBuff(BUFF_ITEM, (WORD)(pItemElem->m_dwItemId));
			REPLACE(g_uIdofMulti, WI_WORLD_MADRIGAL, D3DXVECTOR3(6971.984F, 100.0F, 3336.884F), REPLACE_FORCE, nDefaultLayer);
		}
		else
	#endif
		{
			if (pItemElem->m_dwKeepTime == 0)
			{
				if (CItemElem* pTicket = FindActiveTicket	(pItemElem->m_dwItemId))
				{
					AddDefinedText(TID_GAME_LIMITED_USE);
					return;
				}
				UpdateItem((BYTE)(pItemElem->m_dwObjId), UI_KEEPTIME, pItemElem->GetProp()->dwAbilityMin);
			}
			CTicketProperty* pProperty = CTicketProperty::GetInstance();

	#ifdef __V18Ticket
			const unsigned long sendVar = std::stoul(GetInput());
			const short layer = -1 * (sendVar & 0xFFFF);
			const unsigned long dwWorldId = sendVar >> 16;
			if (dwWorldId != 1)
			{
				if (HasBuffByIk3(pItemProp->dwItemKind3))
				{
					if (!HasBuff(BUFF_ITEM, static_cast<WORD>(pItemElem->m_dwItemId)))
					{
						AddDefinedText(TID_GAME_MUST_STOP_OTHER_TICKET);
						return;
					}
					if (pItemElem->m_dwKeepTime == 0)
					{
						AddDefinedText(TID_GAME_LIMITED_USE);
						return;
					}
					RemoveBuff(BUFF_ITEM, (WORD)pItemElem->m_dwItemId);
					REPLACE(g_uIdofMulti, WI_WORLD_MADRIGAL, D3DXVECTOR3(6971.984F, 100.0F, 3336.884F), REPLACE_FORCE, nDefaultLayer);
					return;
				}
			}

			const TicketProp* tmp = nullptr;
			if (pProperty->IsMultiWorldTicket(pItemElem->m_dwItemId))
			{
				for (const auto it : pProperty->GetMultiworldTicketProp(pItemElem->m_dwItemId)) {
					if (dwWorldId == it->dwWorldId) {
						tmp = it;
						break;
					}
				}
			}
			else
				tmp = pProperty->GetTicketProp(pItemElem->m_dwItemId);

			if (!tmp)
			{
				AddDefinedText(TID_GAME_LIMITED_USE);
				return;
			}

			if (const int nExpand = pProperty->GetExpandedLayer(dwWorldId); layer <= 0 && layer >= -nExpand)
			{
				D3DXVECTOR3 location = 
	#ifdef __ChannelSwitcher
					GetWorld() && GetWorld()->GetID() == dwWorldId ? m_vPos : 
	#endif
					tmp->m_vPos;

				DoApplySkill(static_cast<CCtrl*>(this), pItemProp, nullptr);
				REPLACE(g_uIdofMulti, dwWorldId, location, REPLACE_NORMAL, layer);
			}

	#else
			TicketProp* pTicketProp = pProperty->GetTicketProp(pItemElem->m_dwItemId);
			if (pTicketProp)
			{
				int nLayer = ::atoi(GetInput());
				int nExpand = CTicketProperty::GetInstance()->GetExpanedLayer(pTicketProp->dwWorldId);
				if (nLayer <= 0 && nLayer >= -nExpand)
				{
					DoApplySkill((CCtrl*)this, pItemProp, NULL);
					REPLACE(g_uIdofMulti, pTicketProp->dwWorldId, pTicketProp->vPos, REPLACE_NORMAL, nLayer);
				}
				}
			}
	#endif
		}
	}
	```
	```cpp
	if ((m_nCount & 15) == 0)
	{
	#ifdef __V18Ticket
			if (const CWorld* const pUserWorld = GetWorld(); pUserWorld && CTicketProperty::GetInstance()->IsInTicketedWorld(pUserWorld->GetID()))
	#else
			if (GetWorld() && CTicketProperty::GetInstance()->IsTarget(GetWorld()->GetID()))
	#endif
	```

- **WndMapEx.h**
	```cpp
		enum ConstructionMode { NORMAL, TELEPORTATION, DESTINATION, HYPERLINK 
	#ifdef __CheckLoad
			, CHECKLOAD
	#endif
		};
	```
	```cpp
	#ifdef __CheckLoad
		void AddCheckLoad(std::vector<D3DXVECTOR3>&& val, const unsigned long worldId);
	private:
		void RenderCheckLoad(C2DRender* p2DRender);
		std::vector<D3DXVECTOR3> checkLoad;
	#endif
	```
- **WndMapEx.cpp**
	```cpp
	#ifdef __CheckLoad
	void CWndMapEx::AddCheckLoad(std::vector<D3DXVECTOR3>&& vec, const 	unsigned long worldId)
	{
		checkLoad = std::move(vec);
		m_bMapComboBoxInitialization = true;
		const unsigned char byLocationID = CContinent::GetInstance()->worldIdToMCD(worldId);

		CWndComboBox* pWndComboBoxMapCategory = static_cast<CWndComboBox*>(GetDlgItem(WIDC_COMBOBOX_MAP_CATEGORY));
		CWndComboBox* pWndComboBoxMapName = static_cast<CWndComboBox*>(GetDlgItem(WIDC_COMBOBOX_MAP_NAME));
		CWndComboBox* pWndComboBoxNpcName = static_cast<CWndComboBox*>(GetDlgItem(WIDC_COMBOBOX_NPC_NAME));
		CWndButton* pWndMonsterInfoCheckBox = (CWndButton*)GetDlgItem(WIDC_BUTTON_MONSTER_INFO);
		if (pWndComboBoxNpcName) { pWndComboBoxNpcName->SetVisible(false); }
		if (pWndMonsterInfoCheckBox) { pWndMonsterInfoCheckBox->SetVisible(false); }

		MapComboBoxDataVector* pMapComboBoxDataVector = prj.m_MapInformationManager.GetMapNameVector();
		for (MapComboBoxDataVector::iterator Iterator = pMapComboBoxDataVector->begin(); Iterator != pMapComboBoxDataVector->end(); ++Iterator)
		{
			CMapComboBoxData* pMapComboBoxData = static_cast<CMapComboBoxData*>(*Iterator);
			if (!pMapComboBoxData)
				continue;

			if (byLocationID == pMapComboBoxData->GetLocationID())
			{
				const DWORD dwMapCategoryID = pMapComboBoxData->GetParentID();

				if (pWndComboBoxMapCategory)
				{
					pWndComboBoxMapCategory->SelectItem(dwMapCategoryID);
					pWndComboBoxMapCategory->SetVisible(false);
				}

				if (pWndComboBoxMapName)
				{
					pWndComboBoxMapName->SelectItem(pMapComboBoxData->GetID());
					pWndComboBoxMapName->SetVisible(false);
				}
				break;
			}
		}
	}

	void CWndMapEx::RenderCheckLoad(C2DRender* p2DRender)
	{
		if (!m_pPartyPCArrowTexture || m_eConstructionMode != CHECKLOAD)
			return;

		for (std::vector<D3DXVECTOR3>::const_iterator it = checkLoad.cbegin(); it != checkLoad.cend(); ++it)
		{
			D3DXVECTOR3 vConvertedPosition(0.0F, 0.0F, 0.0F);
			ConvertPosition(vConvertedPosition, *it, m_bySelectedMapLocationID);
			const CPoint pointConvertedPosition(static_cast<int>(vConvertedPosition.x), static_cast<int>(vConvertedPosition.z));

			D3DXVECTOR3 vView = *it;
			vView.y = 0.0F;
			D3DXVec3Normalize(&vView, &vView);

			D3DXVECTOR3 vDatumLine = D3DXVECTOR3(0.0f, 0.0f, 1.0f);
			const float fTheta = D3DXVec3Dot(&vDatumLine, &vView);
			D3DXVECTOR3 vAxis(0.0F, 0.0F, 0.0F);
			D3DXVec3Cross(&vAxis, &vDatumLine, &vView);
			const FLOAT fResultRadian = (vAxis.y >= 0.0F) ? acosf(fTheta) : -acosf(fTheta);
			const FLOAT fRevisedArrowStartPositionX = static_cast<FLOAT>(m_pPCArrowTexture->m_size.cx) * m_fRevisedMapSizeRatio * 0.5F;
			const FLOAT fRevisedArrowStartPositionY = static_cast<FLOAT>(m_pPCArrowTexture->m_size.cy) * m_fRevisedMapSizeRatio * 0.5F;
			const int nRevisedPCArrowStartPositionX = static_cast<int>(static_cast<FLOAT>(pointConvertedPosition.x) - fRevisedArrowStartPositionX);
			const int nRevisedPCArrowStartPositionY = static_cast<int>(static_cast<FLOAT>(pointConvertedPosition.y) - fRevisedArrowStartPositionY);
			m_pPartyPCArrowTexture->RenderRotate(p2DRender, CPoint(nRevisedPCArrowStartPositionX, nRevisedPCArrowStartPositionY), fResultRadian, TRUE, 255, m_fRevisedMapSizeRatio, m_fRevisedMapSizeRatio);
		}
	}
	#endif
	```
	*ondraw*
	```cpp
		RenderUserMarkPosition(p2DRender);
		RenderHyperlinkMarkPosition(p2DRender);
	#ifdef __CheckLoad
		RenderCheckLoad(p2DRender);
	#endif
	```
	*process*
	```cpp
		UpdateDestinationPosition();

	#ifdef __CheckLoad
		if (m_eConstructionMode == CHECKLOAD)
			return true;
	#endif
	```


## License
---
_FlyFF is an mmorpg created by GalaNet/Aeonsoft. This implementation was done by myself:_
| Account  | Site |
| -------- | ------- |
| Kiaemi. | Discord |
| Velvet | RageZone |
| M¿dScientist | Elitepvpers |

