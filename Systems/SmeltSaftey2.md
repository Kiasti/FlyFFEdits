# Smelt Safety Changes
---

[![Smelt Saftey](http://img.youtube.com/vi/qu5mRnuIjYk/0.jpg)](http://www.youtube.com/watch?v=qu5mRnuIjYk)


## Features
---
- Smaller window.
- Displays Tries and breaks them down into the amount of successes and failures.
- One window for any upgrading.(Element, piercing, ultimate, accessory, normal)
- Delay check on the server to detect if a user is abusing the wait period.
- Maximum inventory full of items will continue to upgrade until target.
- Target to + updating at anytime.
- Could Set different Timers for different types (Have to ask for extention if you need it)

## Adding parts to the source
---
- **User.h**
	```cpp
	enum class AttemptType : unsigned char
	{
		DistanceIncorrect,
		MeleeAttack_IncorrectHitNum,
		Timer_CheckLoad,
		Timer_Upgrader,
		Max
	};

	class Sussy
	{
		std::unordered_map<AttemptType, unsigned long> Attempts;
		public:
			unsigned long& GetAttempts(AttemptType&& at) {
				return Attempts[at];
			}
			unsigned long& operator[](AttemptType&& at) {
				return Attempts[at];
			}
	};
	```
	```cpp
		virtual	~CUser();	
		Sussy sus;
	```

- **Msghdr.h**
    ```CPP
    #ifdef __SmeltSafety2
    #define PACKETTYPE_PRIME_SMELT						static_cast<unsigned long>(0x70007004)
    #endif
    ```
- **DpClient.cpp** - neuz
    ```CPP
    #ifdef __SmeltSafety2
    #include "WndNewUpgrade.h"
    #endif
    ```
    find and replace
    ```CPP	
	#ifdef __SmeltSafety2
	void CDPClient::PrimeSmelting()
	{
		BEFORESENDSOLE(ar, PACKETTYPE_PRIME_SMELT, DPID_UNKNOWN);
		SEND(ar, this, DPID_SERVERPLAYER);
	}

	void CDPClient::SendNewSmelt(const unsigned short objid, const unsigned short objid2, const unsigned short objid3, const unsigned short objid4)
	{
		BEFORESENDSOLE(ar, PACKETTYPE_SMELT_SAFETY, DPID_UNKNOWN);
		ar << objid << objid2 << objid3 << objid4;
		SEND(ar, this, DPID_SERVERPLAYER);
	}

	void CDPClient::OnSmeltSafety(CAr& ar)
	{
		unsigned char nResult = 0;
		ar >> nResult;

		Kia::CWndNewUpgrade* pWndSmeltSafety = g_WndMng.m_pWndSmeltSafety;
		if (pWndSmeltSafety == nullptr)
			return;

		pWndSmeltSafety->UpdateReturn(nResult);
	}
	#else
	void CDPClient::SendSmeltSafety(OBJID objid, OBJID objMaterialId, OBJID objProtScrId, OBJID objSmeltScrId)
	{
		BEFORESENDSOLE(ar, PACKETTYPE_SMELT_SAFETY, DPID_UNKNOWN);
		ar << objid << objMaterialId << objProtScrId << objSmeltScrId;
		SEND(ar, this, DPID_SERVERPLAYER);
	}

	void CDPClient::OnSmeltSafety(CAr& ar)
	{
		BYTE nResult = 0;
		ar >> nResult;

		CWndSmeltSafety* pWndSmeltSafety = g_WndMng.m_pWndSmeltSafety;
		if (pWndSmeltSafety == NULL)
			return;

		if (pWndSmeltSafety->m_eWndMode == CWndSmeltSafety::WND_UNIQUE || pWndSmeltSafety->m_eWndMode == CWndSmeltSafety::WND_ULTIMATE)
		{
			switch (nResult)
			{
			case 0:
			{
				pWndSmeltSafety->SetResult(1);
				pWndSmeltSafety->Destroy();
				g_WndMng.PutString(prj.GetText(TID_GAME_TRANSFORM_S00));
				break;
			}
			case 1:
			{
				pWndSmeltSafety->SetResult(2);
				pWndSmeltSafety->AddFailure();
				pWndSmeltSafety->RefreshText();
				break;
			}
			case 2:
			{
				pWndSmeltSafety->Destroy();
				g_WndMng.PutString(prj.GetText(TID_UPGRADE_ERROR_WRONGUPLEVEL));
				break;
			}
			}
		}
		else
		{
			switch (nResult)
			{
			case 1:
			{
				pWndSmeltSafety->DisableScroll2();
				pWndSmeltSafety->SetResult(1);
				pWndSmeltSafety->AddSuccess();
				pWndSmeltSafety->RefreshText();
				break;
			}
			case 2:
			{
				pWndSmeltSafety->SetResult(2);
				pWndSmeltSafety->AddFailure();
				pWndSmeltSafety->RefreshText();
				break;
			}
			}
		}
	}
	#endif
    ```  

- **DPClient.h** _(neuz)_
    ```CPP
	#ifdef __SmeltSafety2
		void PrimeSmelting();
		void SendNewSmelt(unsigned short objid, unsigned short objid2, unsigned short objid3, unsigned short objid4);
	#else
		void SendSmeltSafety(OBJID objid, OBJID objMaterialId, OBJID objProtScrId, OBJID objSmeltScrId = NULL_ID);
	#endif
    ```

- **DpSrvr.cpp** _(Worldserver)_
    find and add
    ```CPP
	#ifdef __SmeltSafety2
		ON_MSG(PACKETTYPE_PRIME_SMELT, &CDPSrvr::OnPrimeSmelt);
	#endif
    ```
    find and change __CDPSrvr::OnSmeltSafety__
    ```CPP
    #if __VER >= 14 // __SMELT_SAFETY
    #ifndef __SmeltSafety2
    void CDPSrvr::OnSmeltSafety( CAr & ar, DPID dpidCache, DPID dpidUser, LPBYTE lpBuf, u_long uBufSize)
    {
    	OBJID dwItemId, dwItemMaterialId, dwItemProtScrId, dwItemSmeltScrId;
    
    	//	pItemSmeltScrId - ÀÏ¹ÝÁ¦·Ã½ÃÀÇ Á¦·ÃµÎ·ç¸¶¸®(»ç¿ë¾ÈÇÒ½Ã¿£ Client¿¡¼­ NULL_ID¸¦ ÀÔ·Â)
    	ar >> dwItemId >> dwItemMaterialId >> dwItemProtScrId >> dwItemSmeltScrId;
    
    	CUser* pUser	= g_UserMng.GetUser( dpidCache, dpidUser );
    
    	if( IsValidObj( pUser ) )
    	//{
    #ifdef __QUIZ
    		if( pUser->GetWorld() && pUser->GetWorld()->GetID() == WI_WORLD_QUIZ )
    		{
    			pUser->AddSmeltSafety( 0 );
    			return;
    		}
    #endif // __QUIZ
    		// Ë¬
    		if( pUser->m_vtInfo.GetOther() || pUser->m_vtInfo.VendorIsVendor() )	// °Å·¡ÁßÀÎ ´ë»óÀÌ ÀÖÀ¸¸é?
    		{
    			pUser->AddSmeltSafety( 0 );
    			return;
    		}
    
    		// ÀÎº¥Åä¸®¿¡ ÀÖ´ÂÁö ÀåÂøµÇ¾î ÀÖ´ÂÁö È®ÀÎÀ» ÇØ¾ß ÇÔ
    		CItemElem* pItemElem0	= pUser->m_Inventory.GetAtId( dwItemId );
    		CItemElem* pItemElem1	= pUser->m_Inventory.GetAtId( dwItemMaterialId );
    		CItemElem* pItemElem2	= pUser->m_Inventory.GetAtId( dwItemProtScrId );
    		CItemElem* pItemElem3	= NULL;
    		if( dwItemSmeltScrId != NULL_ID )
    		{
    			pItemElem3	= pUser->m_Inventory.GetAtId( dwItemSmeltScrId );
    			if( !IsUsableItem( pItemElem3 ) )
    				return;
    		}
    
    		if( IsUsableItem( pItemElem0 ) == FALSE || IsUsableItem( pItemElem1 ) == FALSE || IsUsableItem( pItemElem2     ) == FALSE )
    		{
    			pUser->AddSmeltSafety( 0 );
    			return;
    		}
    
    		// ÀåÂøµÇ¾î ÀÖ´Â ¾ÆÀÌÅÛÀº Á¦·Ã ¸øÇÔ
    		if( pUser->m_Inventory.IsEquip( dwItemId ) )
    		{
    			pUser->AddSmeltSafety( 0 );
    			return;
    		}
    
    		if( pItemElem0->m_nResistSMItemId != 0 ) // »ó¿ëÈ­ ¾ÆÀÌÅÛ Àû¿ëÁßÀÌ¸é ºÒ°¡´É
    		{
    			pUser->AddSmeltSafety( 0 );
    			return;
    		}
    		
    		BYTE nResult = CItemUpgrade::GetInstance()->OnSmeltSafety( pUser, pItemElem0, pItemElem1, pItemElem2,     pItemElem3 );
    
    		pUser->AddSmeltSafety( nResult );
    	}
    }
    #endif
	```

	```cpp
	#ifdef __SmeltSafety2
	#include "../_Interface/WndNewUpgrade.h"
	void CDPSrvr::OnSmeltSafety(CAr& ar, DPID dpidCache, DPID dpidUser, LPBYTE lpBuf, u_long uBufSize)
	{
		unsigned short mainItemOID, matOID, scrollOID, scroll2OID;
		ar >> mainItemOID >> matOID >> scrollOID >> scroll2OID;

		if (CUser* pUser = g_UserMng.GetUser(dpidCache, dpidUser); IsValidObj(pUser))
		{
			if (pUser->m_vtInfo.GetOther() || pUser->m_vtInfo.VendorIsVendor() || pUser->m_Inventory.IsEquip(mainItemOID))
			{
				pUser->AddSmeltSafety(0);
				pUser->AddText("Unable to smelt in a vendor window or the item is equiped.");
				return;
			}

			CItemElem* itemElem = pUser->m_Inventory.GetAtId(mainItemOID);
			CItemElem* matItem = pUser->m_Inventory.GetAtId(matOID);
			CItemElem* scrollItem = pUser->m_Inventory.GetAtId(scrollOID);
			if (!IsUsableItem(itemElem) || !IsUsableItem(matItem) || !IsUsableItem	(scrollItem))
			{
				pUser->AddSmeltSafety(0);
				pUser->AddText("Invalid items. If this continues to happen, please report it to staff");
				return;
			}

			CItemElem* scrollItem2 = pUser->m_Inventory.GetAtId(scroll2OID);
			if (!IsUsableItem(scrollItem2))
				scrollItem2 = nullptr;

			if (auto& timer = pUser->sus[AttemptType::Timer_Upgrader]; g_tmCurrent >= timer)
			{
				timer = g_tmCurrent + Kia::GetUpgradeTimerT() - 200;
				pUser->AddSmeltSafety(CItemUpgrade::GetInstance()->OnSmeltSafety(pUser, itemElem, matItem, scrollItem, scrollItem2));
			}
			else
				pUser->AddSmeltSafety(0);
		}
	}

	void CDPSrvr::OnPrimeSmelt(CAr& ar, DPID dpidCache, DPID dpidUser, LPBYTE lpBuf, u_long uBufSize)
	{
		if (CUser* pUser = g_UserMng.GetUser(dpidCache, dpidUser); IsValidObj(pUser))
		{
			if (pUser->m_vtInfo.GetOther() || pUser->m_vtInfo.VendorIsVendor())
			{
				pUser->AddSmeltSafety(0);
				return;
			}
			pUser->sus[AttemptType::Timer_Upgrader] = g_tmCurrent + Kia::GetUpgradeTimerT() - 200; // -200ms
		}
	}
	#endif
    ```
- **DpSrvr.h** _(worldserver)_
    ```CPP
    #if __VER >= 14 // __SMELT_SAFETY
    	void	OnSmeltSafety( CAr & ar, DPID dpidCache, DPID dpidUser, LPBYTE lpBuf, u_long uBufSize);
    #ifdef __SmeltSafety2
    	void	onPrimeSmelt(CAr& ar, DPID dpidCache, DPID dpidUser, LPBYTE lpBuf, u_long uBufSize);
    #endif
    ```
- **WndField.cpp**
    find and replace in __CWndInventory::OnChildNotify__
    ```CPP
   #ifdef __SmeltSafety2
    							kia::CWndNewUpgrade* pWndSmeltSafety = (kia::CWndNewUpgrade*)GetWndBase(APP_SMELT_SAFETY);
    							pWndSmeltSafety->setItem((CItemElem*)pFocusItem);
    #else
    							CWndSmeltSafety* pWndSmeltSafety = (CWndSmeltSafety*)GetWndBase( APP_SMELT_SAFETY);
    							pWndSmeltSafety->SetItem((CItemElem*)pFocusItem);
    #endif 
    ```
> Note: If you use Florist base, the above code is in CWndMgr::UseItem_MaterialWnd, just look up the APP_.


- **WndManager.h**
    include at top
    ```CPP
    #ifdef __SmeltSafety2
    #include "WndNewUpgrade.h"
    #endif
    ```
    find and replace
    ```CPP
    #ifdef __SmeltSafety2
        kia::CWndNewUpgrade* m_pWndSmeltSafety;
    #else
        CWndSmeltSafety* m_pWndSmeltSafety;
    #endif 
    ```
- **WndWorld.cpp**
    Find and replace in __CWndWorld::OnCommand__
    ```CPP
        #if __VER >= 14 // __SMELT_SAFETY
		#ifdef __SmeltSafety2
				case MMI_SMELT_SAFETY_ACCESSORY:
				case MMI_SMELT_SAFETY_PIERCING:
				case MMI_SMELT_SAFETY_ELEMENT:
		#endif
        		case MMI_SMELT_SAFETY_GENERAL:
        			{
        				if( CMover::GetActiveMover()->m_vtInfo.GetOther() || CMover::GetActiveMover()->m_vtInfo.VendorIsVendor() )
        				{
        					g_WndMng.PutString( prj.GetText(TID_GAME_SMELT_SAFETY_ERROR16), NULL,         prj.GetTextColor(TID_GAME_SMELT_SAFETY_ERROR16) );
        					break;
        				}
        
        				if(g_WndMng.m_pWndSmeltSafety != NULL)
        					SAFE_DELETE(g_WndMng.m_pWndSmeltSafety);

        #ifdef __SmeltSafety2
        				g_WndMng.m_pWndSmeltSafety = new kia::CWndNewUpgrade;
        #else
        				g_WndMng.m_pWndSmeltSafety = new CWndSmeltSafety(CWndSmeltSafety::WND_NORMAL);
        #endif
        				if(g_WndMng.m_pWndSmeltSafety != NULL)
        					g_WndMng.m_pWndSmeltSafety->Initialize();
        				break;
        			}
        #ifndef __SmeltSafety2
        		case MMI_SMELT_SAFETY_ACCESSORY:
        			{
        				if( CMover::GetActiveMover()->m_vtInfo.GetOther() || CMover::GetActiveMover()->m_vtInfo.VendorIsVendor() )
        				{
        					g_WndMng.PutString( prj.GetText(TID_GAME_SMELT_SAFETY_ERROR16), NULL,         prj.GetTextColor(TID_GAME_SMELT_SAFETY_ERROR16) );
        					break;
        				}
        
        				if(g_WndMng.m_pWndSmeltSafety != NULL)
        					SAFE_DELETE(g_WndMng.m_pWndSmeltSafety);
        				g_WndMng.m_pWndSmeltSafety = new CWndSmeltSafety(CWndSmeltSafety::WND_ACCESSARY);
        				if(g_WndMng.m_pWndSmeltSafety != NULL)
        					g_WndMng.m_pWndSmeltSafety->Initialize(NULL);
        				break;
        			}
        		case MMI_SMELT_SAFETY_PIERCING:
        			{
        				if( CMover::GetActiveMover()->m_vtInfo.GetOther() ||         CMover::GetActiveMover()->m_vtInfo.VendorIsVendor() )
        				{
        					g_WndMng.PutString( prj.GetText(TID_GAME_SMELT_SAFETY_ERROR16), NULL,         prj.GetTextColor(TID_GAME_SMELT_SAFETY_ERROR16) );
        					break;
        				}
        
        				if(g_WndMng.m_pWndSmeltSafety != NULL)
        					SAFE_DELETE(g_WndMng.m_pWndSmeltSafety);
        				g_WndMng.m_pWndSmeltSafety = new CWndSmeltSafety(CWndSmeltSafety::WND_PIERCING);
        				if(g_WndMng.m_pWndSmeltSafety != NULL)
        					g_WndMng.m_pWndSmeltSafety->Initialize(NULL);
        				break;
        			}
        #endif
        #endif // __SMELT_SAFETY
        #if __VER >= 15 // __15_5TH_ELEMENTAL_SMELT_SAFETY
        #ifndef __SmeltSafety2
        		case MMI_SMELT_SAFETY_ELEMENT:
        			{
        				if( CMover::GetActiveMover()->m_vtInfo.GetOther() ||         CMover::GetActiveMover()->m_vtInfo.VendorIsVendor() )
        				{
        					g_WndMng.PutString( prj.GetText( TID_GAME_SMELT_SAFETY_ERROR16 ), NULL, prj.GetTextColor(      TID_GAME_SMELT_SAFETY_ERROR16 ) );
        					break;
        				}
        				if( g_WndMng.m_pWndSmeltSafety )
        					SAFE_DELETE( g_WndMng.m_pWndSmeltSafety );
        				g_WndMng.m_pWndSmeltSafety = new CWndSmeltSafety( CWndSmeltSafety::WND_ELEMENT );
        				if( g_WndMng.m_pWndSmeltSafety )
        					g_WndMng.m_pWndSmeltSafety->Initialize( NULL );
        				break;
        			}
        #endif
        #endif // __15_5TH_ELEMENTAL_SMELT_SAFETY
    ```

- **WndNewUpgrade.h** (Create and add to neuz project only)
    ```CPP
	#pragma once
	#if defined(__CLIENT)
	#include "WndNeuz.h"
	#include "WndControl.h"
	#endif

	#if defined(__SmeltSafety2)
	namespace Kia
	{
		enum class EUpgradeTimer : time_t {
			Default = SEC(1), //2
			Quick = SEC(1),
			Long = SEC(1) //4
		};
	
		template <EUpgradeTimer ee = EUpgradeTimer::Default> [[nodiscard]] constexpr time_t GetUpgradeTimerT() noexcept {
			return static_cast<std::underlying_type_t<EUpgradeTimer>>(ee);
		}

		//[[nodiscard]] inline time_t GetUpgradeTimer(EUpgradeTimer&& ee) {
		//	return static_cast<std::underlying_type_t<EUpgradeTimer>>(ee);
		//}
	#if defined(__CLIENT)
		enum class EUpgradeSetting : unsigned char {
			none, normal, ultimate, jewel, element, piercing, baruna
		};

		class CWndNewUpgrade final : public CWndNeuz
		{
			EUpgradeSetting setting;
			CItemElem* itemElem;
			CTexture* itemTex, * matTex, * scrollTex, * scroll2Tex;

			unsigned long matId, scrollId, scroll2Id;
			unsigned long matCount, scrollCount, scrollCount2;

			LPDIRECT3DVERTEXBUFFER9 m_pVertexBufferGauge, 	m_pVertexBufferSuccessImage, m_pVertexBufferFailureImage;
			CTexture* m_pNowGaugeTexture, * m_pSuccessTexture, * m_pFailureTexture;
			unsigned long enchantStart, enchantEnd;
			float m_fGaugeRate;
			int failures, successes;

			CWndStatic* currentUpgradeStatic, * currentSettingStatic;
			CWndButton* startbutton, * endButton, * resetButton;
			CWndEdit* maxLimitEdit;

			unsigned long maxValue;
			unsigned char result;
			bool start;

			public:
				CWndNewUpgrade();
				~CWndNewUpgrade() override;

				CWndNewUpgrade(CWndNewUpgrade const&) = delete;
				CWndNewUpgrade& operator =(CWndNewUpgrade const&) = delete;
				CWndNewUpgrade(CWndNewUpgrade&&) = delete;
				CWndNewUpgrade& operator=(CWndNewUpgrade&&) = delete;

				[[nodiscard]] unsigned long GetDefaultMax() const;
				[[nodiscard]] unsigned long GetCurValue() const;
				void UpdateUpgradeStatic() const;
				void ResetMats();

				bool SetItem(CItemElem* item);
				void CheckScroll();
				void CheckScroll2();
				void UpdateReturn(unsigned char value = 0);
				static CItemElem* GetFirstOfItemId(unsigned long itemId);

				BOOL OnDropIcon(LPSHORTCUT pShortcut, CPoint point) override;
				void OnLButtonUp(UINT nFlags, CPoint point) override;
				BOOL Initialize(CWndBase* pWndParent = nullptr, DWORD dwWndId = 0) override;
				BOOL OnChildNotify(UINT message, UINT nID, LRESULT* pLResult) override;
				void OnDraw(C2DRender* p2DRender) override;
				void OnInitialUpdate() override;
				BOOL Process() override;
				HRESULT RestoreDeviceObjects() override;
				HRESULT InvalidateDeviceObjects() override;
				HRESULT DeleteDeviceObjects() override;


				/**
				 *	@brief	Generate function based on compile time to set mode.
				*/
				template <EUpgradeSetting ups = EUpgradeSetting::none> void SetMode()
				{
					if constexpr(ups == EUpgradeSetting::normal)
					{
						setting = EUpgradeSetting::normal;
						SetTitle("Normal Upgrade");
						currentSettingStatic->SetTitle("General Upgrading");
					}
					else if constexpr (ups == EUpgradeSetting::ultimate)
					{
						setting = EUpgradeSetting::ultimate;
						SetTitle("Ultimate Upgrade");
						currentSettingStatic->SetTitle("Ultimate Upgrading");
					}
					else if constexpr (ups == EUpgradeSetting::jewel)
					{
						setting = EUpgradeSetting::jewel;
						SetTitle("Accessory Upgrade");
						currentSettingStatic->SetTitle("Accessory Upgrading");
					}
					else if constexpr (ups == EUpgradeSetting::piercing)
					{
						setting = EUpgradeSetting::piercing;
						SetTitle("Piercing");
						currentSettingStatic->SetTitle("Piercing");
					}
					else if constexpr (ups == EUpgradeSetting::element)
					{
						setting = EUpgradeSetting::element;
						SetTitle("Element Upgrade");
						currentSettingStatic->SetTitle("Element Upgrade");
					}
					else if constexpr (ups == EUpgradeSetting::none)
					{
						setting = EUpgradeSetting::none;
						SetTitle("Upgrade");
						currentSettingStatic->SetTitle("Upgrade");
					}
				}


		};
	#endif
	}
	#endif
    ```
- **WndNewUpgrade.cpp** (create and add to neuz project)
   ```cpp
	#include "StdAfx.h"
	#if defined(__CLIENT) && defined(__SmeltSafety2)
	#include "WndNewUpgrade.h"
	#include "path.h"
	#include "ResData.h"
	#include "DPClient.h"
	extern CDPClient g_DPlay;
	
	
	Kia::CWndNewUpgrade::CWndNewUpgrade() : setting(EUpgradeSetting::none), itemElem(nullptr), itemTex(nullptr), matTex(nullptr), scrollTex(nullptr), scroll2Tex(nullptr), matId(0), scrollId(0), scroll2Id(0), matCount(0), scrollCount(0),
		scrollCount2(0), m_pVertexBufferGauge(nullptr), m_pVertexBufferSuccessImage(nullptr), m_pVertexBufferFailureImage(nullptr), m_pNowGaugeTexture(nullptr), m_pSuccessTexture(nullptr), m_pFailureTexture(nullptr),
		enchantStart(0), enchantEnd(0), m_fGaugeRate(0), failures(0), successes(0), currentUpgradeStatic(nullptr), currentSettingStatic(nullptr), startbutton(nullptr), endButton(nullptr), resetButton(nullptr),
		maxLimitEdit(nullptr), maxValue(20), result(0), start(false)
	{
	}
	
	Kia::CWndNewUpgrade::~CWndNewUpgrade()
	{
		if (itemElem)
		{
			itemElem->SetExtra(itemElem->GetExtra() - 1);
			itemElem = nullptr;
			itemTex = nullptr;
		}
		ResetMats();
	
		if (CWndInventory* pWndInventory = dynamic_cast<CWndInventory*>(GetWndBase(APP_INVENTORY)))
			pWndInventory->m_wndItemCtrl.SetDieFlag(false);
	}
	
	BOOL Kia::CWndNewUpgrade::Initialize(CWndBase* pWndParent, DWORD)
	{
		return InitDialog(g_Neuz.GetSafeHwnd(), APP_SMELT_SAFETY, 0, CPoint(0, 0), pWndParent);
	}
	
	void Kia::CWndNewUpgrade::OnInitialUpdate()
	{
		CWndNeuz::OnInitialUpdate();
		startbutton = dynamic_cast<CWndButton*>(GetDlgItem(WIDC_BUTTON_START));
		endButton = dynamic_cast<CWndButton*>(GetDlgItem(WIDC_BUTTON_STOP));
		resetButton = dynamic_cast<CWndButton*>(GetDlgItem(WIDC_BUTTON_RESET));
	
		maxLimitEdit = dynamic_cast<CWndEdit*>(GetDlgItem(WIDC_EDIT_MAX_GRADE));
		maxLimitEdit->AddWndStyle(EBS_READONLY);
		maxLimitEdit->SetString(std::to_string(maxValue).c_str());
	
		currentSettingStatic = dynamic_cast<CWndStatic*>(GetDlgItem(WIDC_TITLE_NOW_GRADE));
		currentUpgradeStatic = dynamic_cast<CWndStatic*>(GetDlgItem(WIDC_NOW_GRADE));
	
		m_pNowGaugeTexture = m_textureMng.AddTexture(g_Neuz.m_pd3dDevice, MakePath(DIR_THEME, "SafetyGauge.bmp"), 0xffff00ff);
		m_pSuccessTexture = m_textureMng.AddTexture(g_Neuz.m_pd3dDevice, MakePath(DIR_THEME, "SafetySuccess.bmp"), 0xffff00ff);
		m_pFailureTexture = m_textureMng.AddTexture(g_Neuz.m_pd3dDevice, MakePath(DIR_THEME, "SafetyFailure.bmp"), 0xffff00ff);
	
		CWndStatic* pWndStatic = dynamic_cast<CWndStatic*>(GetDlgItem(WIDC_TITLE_MAX_GRADE));
		pWndStatic->SetTitle("Maximum");
		MoveParentCenter();
	
		if (CWndInventory* pWndInventory = dynamic_cast<CWndInventory*>(g_WndMng.CreateApplet(APP_INVENTORY)))
			pWndInventory->m_wndItemCtrl.SetDieFlag(true);
	}
	
	CItemElem* Kia::CWndNewUpgrade::GetFirstOfItemId(const unsigned long itemId) {
		return g_pPlayer ? g_pPlayer->m_Inventory.GetAtItemId(itemId) : nullptr;
	}
	
	unsigned long Kia::CWndNewUpgrade::GetCurValue() const
	{
		if (itemElem && itemElem->GetProp())
		{
			switch (setting)
			{
				case EUpgradeSetting::normal:
				case EUpgradeSetting::ultimate:
				case EUpgradeSetting::baruna:
				case EUpgradeSetting::jewel:
					return itemElem->GetAbilityOption();
				case EUpgradeSetting::element:
					return itemElem->m_nResistAbilityOption;
				case EUpgradeSetting::piercing:
					return itemElem->GetPiercingSize();
				case EUpgradeSetting::none:
					break;
			}		
		}
		return 0;
	}
	
	BOOL Kia::CWndNewUpgrade::Process()
	{
		if (CMover::GetActiveMover()->m_vtInfo.GetOther() || CMover::GetActiveMover()->m_vtInfo.VendorIsVendor())
			Destroy();
	
		if (start)
		{
			m_fGaugeRate = static_cast<float>(g_tmCurrent - enchantStart) / static_cast<float>(enchantEnd - enchantStart);
			if (enchantEnd < g_tmCurrent)
			{
				enchantStart = 0xffffffff;
				enchantEnd = 0xffffffff;
	
				const CItemElem* ptr = GetFirstOfItemId(matId);
				const CItemElem* ptr2 = GetFirstOfItemId(scrollId);
				const CItemElem* ptr3 = GetFirstOfItemId(scroll2Id);
				g_DPlay.SendNewSmelt(static_cast<unsigned short>(itemElem->m_dwObjId), ptr ? static_cast<unsigned short>(ptr->m_dwObjId) : -1, 
							ptr2 ? static_cast<unsigned short>(ptr2->m_dwObjId) : -1, ptr3 ? static_cast<unsigned short>(ptr3->m_dwObjId) : -1);
			}
			startbutton->EnableWindow(false);
			endButton->EnableWindow(true);
			resetButton->EnableWindow(false);
			if (GetCurValue() >= maxValue || GetCurValue() >= GetDefaultMax())
			{
				start = false;
				enchantStart = 0xffffffff;
				enchantEnd = 0xffffffff;
			}
		}
		else
		{
			m_fGaugeRate = 0.0f;
			enchantStart = 0xffffffff;
			enchantEnd = 0xffffffff;
			if (!itemElem)
				startbutton->EnableWindow(false);
			else if (matCount && scrollCount)
				startbutton->EnableWindow(true);
	
			endButton->EnableWindow(false);
			resetButton->EnableWindow(true);
		}
		return true;
	}
	
	void Kia::CWndNewUpgrade::ResetMats()
	{
		if (matId != 0)
		{
			for (unsigned long slotId = 0; slotId < g_pPlayer->m_Inventory.GetMax(); ++slotId)
			{
				if (CItemElem* pItemElem = &g_pPlayer->m_Inventory.m_apItem[slotId]; pItemElem && matId == pItemElem->m_dwItemId)
					pItemElem->SetExtra(0);
			}
			matCount = 0;
			matId = 0;
			matTex = nullptr;
		}
		if (scrollId != 0)
		{
			for (unsigned long slotId = 0; slotId < g_pPlayer->m_Inventory.GetMax(); ++slotId)
			{
				if (CItemElem* pItemElem = &g_pPlayer->m_Inventory.m_apItem[slotId]; pItemElem && scrollId == pItemElem->m_dwItemId)
					pItemElem->SetExtra(0);
			}
			scrollCount = 0;
			scrollId = 0;
			scrollTex = nullptr;
		}
		if (scroll2Id != 0)
		{
			for (unsigned long slotId = 0; slotId < g_pPlayer->m_Inventory.GetMax(); ++slotId)
			{
				if (CItemElem* pItemElem = &g_pPlayer->m_Inventory.m_apItem[slotId]; pItemElem && scroll2Id == pItemElem->m_dwItemId)
					pItemElem->SetExtra(0);
			}
			scrollCount2 = 0;
			scroll2Id = 0;
			scroll2Tex = nullptr;
		}
		setting = EUpgradeSetting::none;
		SetTitle("Safe Upgrade");
		currentSettingStatic->SetTitle("Upgrade");
		result = 0;
	}
	
	void Kia::CWndNewUpgrade::CheckScroll()
	{
		unsigned long tempScrollId = NULL_ID;
		switch (setting)
		{
			case EUpgradeSetting::normal:
			case EUpgradeSetting::element:
				tempScrollId = II_SYS_SYS_SCR_SMELPROT; 
				break;
			case EUpgradeSetting::jewel:
				tempScrollId = II_SYS_SYS_SCR_SMELPROT4; 
				break;
			case EUpgradeSetting::piercing:
				tempScrollId = II_SYS_SYS_SCR_PIEPROT; 
				break;
			case EUpgradeSetting::ultimate:
				tempScrollId = II_SYS_SYS_SCR_SMELPROT3; 
				break;
			case EUpgradeSetting::baruna:
			case EUpgradeSetting::none:
				tempScrollId = NULL_ID;
				break;
		}
	
		if (scrollId != 0 && tempScrollId != scrollId)
		{
			scrollCount = 0;
			for (unsigned long slotId = 0; slotId < g_pPlayer->m_Inventory.GetMax(); ++slotId)
			{
				if (CItemElem* pItemElem = &g_pPlayer->m_Inventory.m_apItem[slotId]; pItemElem && scrollId == pItemElem->m_dwItemId)
					if (pItemElem->GetExtra())
						pItemElem->SetExtra(0);
			}
			scrollId = 0;
			scrollTex = nullptr;
		}
		CheckScroll2();
	}
	
	
	void Kia::CWndNewUpgrade::CheckScroll2()
	{
		unsigned long tempScrollId = NULL_ID;
		switch (setting)
		{
			case EUpgradeSetting::normal:
				tempScrollId = II_SYS_SYS_SCR_SMELTING; 
				break;
			case EUpgradeSetting::element:
				tempScrollId = II_SYS_SYS_SCR_SMELTING2;  
				break;
			case EUpgradeSetting::baruna:
			case EUpgradeSetting::piercing:
			case EUpgradeSetting::ultimate:
			case EUpgradeSetting::jewel:
			case EUpgradeSetting::none:
				tempScrollId = NULL_ID;
				break;
		}
	
		if (scroll2Id != 0 && tempScrollId != scroll2Id)
		{
			scrollCount2 = 0;
			for (unsigned long slotId = 0; slotId < g_pPlayer->m_Inventory.GetMax(); ++slotId)
			{
				if (CItemElem* pItemElem = &g_pPlayer->m_Inventory.m_apItem[slotId]; pItemElem && scroll2Id == pItemElem->m_dwItemId)
					if (pItemElem->GetExtra())
						pItemElem->SetExtra(0);
			}
			scroll2Id = 0;
			scroll2Tex = nullptr;
		}
	}
	
	
	void Kia::CWndNewUpgrade::UpdateUpgradeStatic() const
	{
		if (currentUpgradeStatic && itemElem)
		{
			CString str = "";
			switch (setting)
			{
				case EUpgradeSetting::normal:
				case EUpgradeSetting::ultimate:
				case EUpgradeSetting::baruna:
				case EUpgradeSetting::jewel:
					str.Format("%d", itemElem->GetAbilityOption());
					break;
				case EUpgradeSetting::element:
					str.Format("%d", itemElem->m_nResistAbilityOption);
					break;
				case EUpgradeSetting::piercing:
					str.Format("%d", itemElem->GetPiercingSize());
					break;
				case EUpgradeSetting::none:
					break;
			}
			currentUpgradeStatic->SetTitle(str);
		}
	}
	
	void Kia::CWndNewUpgrade::OnLButtonUp(UINT nFlags, const CPoint point)
	{
		if (start)
			return;
	
		CWndStatic* wndStatic = dynamic_cast<CWndStatic*>(GetDlgItem(WIDC_STATIC1));
		if (wndStatic && wndStatic->GetWndRect().PtInRect(point))
		{
			if (itemElem)
			{
				itemElem->SetExtra(itemElem->GetExtra() - 1);
				itemElem = nullptr;
				itemTex = nullptr;
				ResetMats();
				successes = 0;
				failures = 0;
			}
		}
	
		wndStatic = dynamic_cast<CWndStatic*>(GetDlgItem(WIDC_STATIC11));
		if (wndStatic && wndStatic->GetWndRect().PtInRect(point))
		{
			if (matId)
				ResetMats();
		}
	
		wndStatic = dynamic_cast<CWndStatic*>(GetDlgItem(WIDC_STATIC41));
		if (wndStatic && wndStatic->GetWndRect().PtInRect(point))
		{
			if (scrollId)
			{
				scrollCount = 0;
				for (unsigned long slotId = 0; slotId < g_pPlayer->m_Inventory.GetMax(); ++slotId)
				{
					if (CItemElem* pItemElem = &g_pPlayer->m_Inventory.m_apItem[slotId]; pItemElem && scrollId == pItemElem->m_dwItemId)
						if (pItemElem->GetExtra())
							pItemElem->SetExtra(0);
				}
				scrollId = 0;
				scrollTex = nullptr;
			}
			result = 0;
		}
	
		wndStatic = dynamic_cast<CWndStatic*>(GetDlgItem(WIDC_STATIC61));
		if (wndStatic && wndStatic->GetWndRect().PtInRect(point))
		{
			if (scroll2Id)
			{
				scrollCount2 = 0;
				for (unsigned long slotId = 0; slotId < g_pPlayer->m_Inventory.GetMax(); ++slotId)
				{
					if (CItemElem* pItemElem = &g_pPlayer->m_Inventory.m_apItem[slotId]; pItemElem && scroll2Id == pItemElem->m_dwItemId)
						if (pItemElem->GetExtra())
							pItemElem->SetExtra(0);
				}
				scroll2Id = 0;
				scroll2Tex = nullptr;
			}
		}
	}
	
	/**
	*	@brief	Sets an item to the window, checking if it is a valid item for the others.
	* 
	*	@param	item The item being added in the upgrade window
	*	@return	True/False if item is added.
	*/
	bool Kia::CWndNewUpgrade::SetItem(CItemElem* item)
	{
		if (start)
			return false;
	
		const ItemProp* iProp = item->GetProp();
		if (iProp == nullptr)
			return false;
	
		const auto oldSetting = setting;
		if (iProp->dwItemKind1 == IK1_WEAPON || iProp->dwItemKind2 == IK2_ARMOR || iProp->dwItemKind2 == IK2_ARMORETC || iProp->dwItemKind2 == IK2_JEWELRY || item->IsCollector())
		{
			if (itemElem)
			{
				itemElem->SetExtra(itemElem->GetExtra() - 1);
				itemElem = nullptr;
				itemTex = nullptr;
			}
			itemElem = item;
			itemElem->SetExtra(itemElem->GetExtra() + 1);
	
			if (matId)
			{
				if (const ItemProp* matProp = prj.GetItemProp(static_cast<int>(matId)))
				{
					bool resetMat = false;
					switch (matProp->dwItemKind3)
					{
						case IK3_SMELT_GENERAL_MATERIAL:
							if (iProp->dwItemKind2 == IK2_JEWELRY || item->IsCollector() || iProp->dwReferStat1 == WEAPON_ULTIMATE)
								resetMat = true;
							break;
						case IK3_SMELT_ULTIMATE_MATERIAL:
							if (iProp->dwReferStat1 != WEAPON_ULTIMATE)
								resetMat = true;
							break;
						case IK3_SMELT_ACCESSORY_MATERIAL:
							if (iProp->dwItemKind2 != IK2_JEWELRY && iProp->dwItemKind3 != IK3_SUIT && iProp->dwItemKind1 != IK1_WEAPON)
								resetMat = true;
							break;
						case IK3_ELECARD:
							if (iProp->dwItemKind1 != IK1_WEAPON && iProp->dwItemKind3 != IK3_SUIT && iProp->dwItemKind3 == IK3_SHIELD)
								resetMat = true;
							break;
						default:
							switch (matProp->dwID)
							{
								case II_GEN_MAT_ORICHALCUM01:
								case II_GEN_MAT_ORICHALCUM01_1:
									if (iProp->dwItemKind2 == IK2_JEWELRY || item->IsCollector() || iProp->dwReferStat1 == WEAPON_ULTIMATE)
										resetMat = true;
									break;
								case II_GEN_MAT_ORICHALCUM02:
									if (iProp->dwReferStat1 != WEAPON_ULTIMATE)
										resetMat = true;
									break;
								case II_GEN_MAT_MOONSTONE:
								case II_GEN_MAT_MOONSTONE_1:
									if (iProp->dwItemKind2 != IK2_JEWELRY && iProp->dwItemKind3 != IK3_SUIT && iProp->dwItemKind1 != IK1_WEAPON && iProp->dwItemKind3 == IK3_SHIELD)
										resetMat = true;
									break;
								default:
									break;
							}
							break;
					}
					if (resetMat)
						ResetMats();
				}
			}
			else
				ResetMats(); // force reset if there is no mat anyway.
	
			itemTex = m_textureMng.AddTexture(g_Neuz.m_pd3dDevice, MakePath(DIR_ITEM, iProp->szIcon), 0xffff00ff);
			SetTitle("Upgrade");
			UpdateUpgradeStatic();
		}
		else if (iProp->dwItemKind3 == IK3_SMELT_GENERAL_MATERIAL || iProp->dwItemKind3 == IK3_SMELT_ULTIMATE_MATERIAL || iProp->dwItemKind3 == IK3_SMELT_ACCESSORY_MATERIAL || iProp->dwItemKind3 == IK3_ELECARD		
			// old resource
				|| iProp->dwItemKind3 == IK3_ENCHANT || iProp->dwItemKind3 == IK3_PIERDICE
		
				)
		{
			if (!itemElem)
				return false;
	
			if (matId == iProp->dwID)
				return false;
	
			ResetMats();
			if (const ItemProp* mainProp = itemElem->GetProp())
			{
				bool set = false;
				switch (iProp->dwItemKind3)
				{
					case IK3_SMELT_GENERAL_MATERIAL:
						if (!itemElem->IsCollector() && mainProp->dwReferStat1 != WEAPON_ULTIMATE && (mainProp->dwItemKind1 == IK1_WEAPON || mainProp->dwItemKind2 == IK2_ARMOR || mainProp->dwItemKind2 == IK2_ARMORETC) )
						{
							SetMode<EUpgradeSetting::normal>();
							set = true;
						}
						break;
					case IK3_SMELT_ULTIMATE_MATERIAL:
						if ((mainProp->dwItemKind1 == IK1_WEAPON || mainProp->dwItemKind2 == IK2_ARMOR || mainProp->dwItemKind2 == IK2_ARMORETC) && mainProp->dwReferStat1 == WEAPON_ULTIMATE)
						{
							SetMode<EUpgradeSetting::ultimate>();
							set = true;
						}
						break;
					case IK3_SMELT_ACCESSORY_MATERIAL:
						if (mainProp->dwItemKind2 == IK2_JEWELRY || itemElem->IsCollector())
						{
							SetMode<EUpgradeSetting::jewel>();
							set = true;
						}
						else if (mainProp->dwItemKind1 == IK1_WEAPON || mainProp->dwItemKind3 == IK3_SUIT || mainProp->dwItemKind3 == IK3_SHIELD)
						{
							SetMode<EUpgradeSetting::piercing>();
							set = true;
						}
						break;
					case IK3_ELECARD:
						if (mainProp->dwItemKind1 == IK1_WEAPON || mainProp->dwItemKind3 == IK3_SUIT || mainProp->dwItemKind3 == IK3_SHIELD)
						{
							SetMode<EUpgradeSetting::element>();
							set = true;
						}
						break;
					default: 
					{
						switch (iProp->dwID)
						{
							case II_GEN_MAT_ORICHALCUM01:
							case II_GEN_MAT_ORICHALCUM01_1:
								if (!itemElem->IsCollector() && mainProp->dwReferStat1 != WEAPON_ULTIMATE && (mainProp->dwItemKind1 == IK1_WEAPON || mainProp->dwItemKind2 == IK2_ARMOR || mainProp->dwItemKind2 == IK2_ARMORETC))
								{
									SetMode<EUpgradeSetting::normal>();
									set = true;
								}
								break;
							case II_GEN_MAT_ORICHALCUM02:
								if ((mainProp->dwItemKind1 == IK1_WEAPON || mainProp->dwItemKind2 == IK2_ARMOR || mainProp->dwItemKind2 == IK2_ARMORETC) && mainProp->dwReferStat1 == WEAPON_ULTIMATE)
								{
									SetMode<EUpgradeSetting::ultimate>();
									set = true;
								}
								break;
						case II_GEN_MAT_MOONSTONE:
						case II_GEN_MAT_MOONSTONE_1:
							if (mainProp->dwItemKind2 == IK2_JEWELRY || itemElem->IsCollector()) {
								SetMode<EUpgradeSetting::jewel>();
								set = true;
							}
							else if (mainProp->dwItemKind1 == IK1_WEAPON || mainProp->dwItemKind3 == IK3_SUIT || mainProp->dwItemKind3 == IK3_SHIELD) {
								SetMode<EUpgradeSetting::piercing>();
								set = true;
							}
							break;
						default:
							break;
						}
						break;
					}
				}
	
				if (set)
				{
					matCount = 0;
					matId = item->m_dwItemId;
					matTex = m_textureMng.AddTexture(g_Neuz.m_pd3dDevice, MakePath(DIR_ITEM, iProp->szIcon), 0xffff00ff);
					for (unsigned long slotId = 0; slotId < g_pPlayer->m_Inventory.GetMax(); ++slotId)
					{
						if (CItemElem* pItemElem = &g_pPlayer->m_Inventory.m_apItem[slotId]; pItemElem && matId == pItemElem->m_dwItemId)
						{
							pItemElem->SetExtra(pItemElem->m_nItemNum);
							matCount += pItemElem->m_nItemNum;
						}
					}
					CheckScroll();
					UpdateUpgradeStatic();
					maxValue = GetDefaultMax();
				}
			}
		}
		else
		{
			if (itemElem && matId)
			{
				const ItemProp* matProp = prj.GetItemProp(static_cast<int>(matId));
				if (ItemProp* mainProp = itemElem->GetProp(); matProp && mainProp)
				{
					bool isNormal = (matProp->dwItemKind3 == IK3_SMELT_GENERAL_MATERIAL && item->m_dwItemId == II_SYS_SYS_SCR_SMELPROT);
					bool isUlti = (matProp->dwItemKind3 == IK3_SMELT_ULTIMATE_MATERIAL && item->m_dwItemId == II_SYS_SYS_SCR_SMELPROT3) && mainProp->IsUltimate();
					bool isPiercing = (mainProp->dwItemKind1 == IK1_WEAPON || mainProp->dwItemKind3 == IK3_SUIT) && II_SYS_SYS_SCR_PIEPROT == item->m_dwItemId && matProp->dwItemKind3 == IK3_SMELT_ACCESSORY_MATERIAL;
					bool isAccessory = matProp->dwItemKind3 == IK3_SMELT_ACCESSORY_MATERIAL && item->m_dwItemId == II_SYS_SYS_SCR_SMELPROT4 && (mainProp->dwItemKind2 == IK2_JEWELRY || itemElem->IsCollector());
					const bool isEle = (matProp->dwItemKind3 == IK3_ELECARD && item->m_dwItemId == II_SYS_SYS_SCR_SMELPROT && (mainProp->dwItemKind1 == IK1_WEAPON || mainProp->dwItemKind3 == IK3_SUIT || mainProp->dwItemKind3 == IK3_SHIELD));
	
					// old resource
					bool ApplyScroll = isNormal || isUlti || isPiercing || isAccessory || isEle;
					if (!ApplyScroll)
					{
						isNormal = item->m_dwItemId == II_SYS_SYS_SCR_SMELPROT && (matProp->dwID == II_GEN_MAT_ORICHALCUM01 || matProp->dwID == II_GEN_MAT_ORICHALCUM01_1);
						isUlti = matProp->dwID == II_GEN_MAT_ORICHALCUM02 && item->m_dwItemId == II_SYS_SYS_SCR_SMELPROT3 && mainProp->IsUltimate();
						isPiercing = (mainProp->dwItemKind1 == IK1_WEAPON || mainProp->dwItemKind3 == IK3_SUIT || mainProp->dwItemKind3 == IK3_SHIELD) && II_SYS_SYS_SCR_PIEPROT == item->m_dwItemId && (matProp->dwID == II_GEN_MAT_MOONSTONE || matProp->dwID == II_GEN_MAT_MOONSTONE_1);
						isAccessory = (matProp->dwID == II_GEN_MAT_MOONSTONE || matProp->dwID == II_GEN_MAT_MOONSTONE_1) && item->m_dwItemId == II_SYS_SYS_SCR_SMELPROT4 && (mainProp->dwItemKind2 == IK2_JEWELRY || itemElem->IsCollector());
						ApplyScroll = isNormal || isUlti || isPiercing || isAccessory || isEle;
					}
					if (ApplyScroll)
					{
						scrollId = item->m_dwItemId;
						scrollTex = m_textureMng.AddTexture(g_Neuz.m_pd3dDevice, MakePath(DIR_ITEM, iProp->szIcon), 0xffff00ff);
						scrollCount = 0;
						for (unsigned long slotId = 0; slotId < g_pPlayer->m_Inventory.GetMax(); ++slotId)
						{
							if (CItemElem* pItemElem = &g_pPlayer->m_Inventory.m_apItem[slotId]; pItemElem && scrollId == pItemElem->m_dwItemId)
							{
								pItemElem->SetExtra(pItemElem->m_nItemNum);
								scrollCount += pItemElem->m_nItemNum;
							}
						}
					}
					else if ((matProp->dwItemKind3 == IK3_SMELT_GENERAL_MATERIAL && II_SYS_SYS_SCR_SMELTING == item->m_dwItemId) || (II_SYS_SYS_SCR_SMELTING2 == item->m_dwItemId && matProp->dwItemKind3 == IK3_ELECARD))
					{
						scroll2Id = item->m_dwItemId;
						scroll2Tex = m_textureMng.AddTexture(g_Neuz.m_pd3dDevice, MakePath(DIR_ITEM, iProp->szIcon), 0xffff00ff);
						scrollCount2 = 0;
						for (unsigned long slotId = 0; slotId < g_pPlayer->m_Inventory.GetMax(); ++slotId)
						{
							if (CItemElem* pItemElem = &g_pPlayer->m_Inventory.m_apItem[slotId]; pItemElem && scroll2Id == pItemElem->m_dwItemId)
							{
								pItemElem->SetExtra(pItemElem->m_nItemNum);
								scrollCount2 += pItemElem->m_nItemNum;
							}
						}
					}
				}
			}
		}
		if (oldSetting != setting)
			maxLimitEdit->SetString(std::to_string(GetDefaultMax()).c_str());
	
		return true;
	}
	
	BOOL Kia::CWndNewUpgrade::OnDropIcon(LPSHORTCUT pShortcut, const CPoint point)
	{
		if (start)
			return false;
	
		CWndBase* pWndFrame = pShortcut->m_pFromWnd->GetFrameWnd();
		if (pWndFrame == nullptr)
			return false;
	
		if (pWndFrame->GetWndId() != APP_INVENTORY)
		{
			SetForbid(true);
			return false;
		}
	
		if (pShortcut->m_dwType != SHORTCUT_ITEM)
			return false;
	
		if (const CRect WndRect = GetClientRect(); WndRect.PtInRect(point))
		{
			CItemElem* pTempElem = g_pPlayer->m_Inventory.GetAtItemId(pShortcut->m_dwId);
			if (pTempElem == nullptr)
				return false;
	
			SetItem(pTempElem);
		}
		return false;
	}
	
	void Kia::CWndNewUpgrade::UpdateReturn(const unsigned char value)
	{
		result = value;
		if (result != 1 && result != 2)
		{
			start = false;
			enchantStart = NULL_ID;
			enchantEnd = NULL_ID;
			if (result == 3)
				g_WndMng.PutString("Failed to upgrade. It's already maxed out");
			return;
		}
	
		if (result == 2)
			failures++;
		else if (result == 1)
			successes++;
	
		std::string tempTitle = "Tries: ";
		tempTitle += std::to_string(failures + successes);
		tempTitle += "\tSucces: ";
		tempTitle += std::to_string(successes);
		tempTitle += "   Failures: ";
		tempTitle += std::to_string(failures);
		SetTitle(tempTitle.c_str());
	
		if (matCount > 0)
			matCount--;
		if (scrollCount > 0)
			scrollCount--;
		if (scrollCount2 > 0)
			scrollCount2--;
	
		if (GetDefaultMax() == GetCurValue() || matCount == 0 || scrollCount == 0)
		{
			start = false;
			enchantStart = NULL_ID;
			enchantEnd = NULL_ID;
			result = 0;
			ResetMats();
		}
		else
		{
			enchantStart = g_tmCurrent;
			enchantEnd = g_tmCurrent + GetUpgradeTimerT<EUpgradeTimer::Default>();
		}
		if (scrollCount2 == 0)
			scroll2Tex = nullptr;
		UpdateUpgradeStatic();
	}
	
	
	unsigned long Kia::CWndNewUpgrade::GetDefaultMax() const
	{
		switch (setting)
		{
			case EUpgradeSetting::normal:
			case EUpgradeSetting::ultimate:
				return 10;
			case EUpgradeSetting::jewel: // MAX_AAO
			case EUpgradeSetting::element:
			case EUpgradeSetting::baruna:
				return 20;
			case EUpgradeSetting::piercing:
				if (itemElem)
				{
					if (const ItemProp* iProp = itemElem->GetProp(); iProp && iProp->dwItemKind3 == IK3_SUIT)
						return MAX_PIERCING_SUIT;
				}
				return MAX_PIERCING;
			case EUpgradeSetting::none:
				return 1;
		}
		return 1;
	}
	
	HRESULT Kia::CWndNewUpgrade::RestoreDeviceObjects()
	{
		CWndBase::RestoreDeviceObjects();
		if (m_pVertexBufferGauge == nullptr)
			m_pApp->m_pd3dDevice->CreateVertexBuffer(sizeof(TEXTUREVERTEX2) * 3 * 6, D3DUSAGE_WRITEONLY | D3DUSAGE_DYNAMIC, D3DFVF_TEXTUREVERTEX2, D3DPOOL_DEFAULT, &m_pVertexBufferGauge, nullptr);
	
		if (m_pVertexBufferSuccessImage == nullptr)
			m_pApp->m_pd3dDevice->CreateVertexBuffer(sizeof(TEXTUREVERTEX2) * 3 * 6, D3DUSAGE_WRITEONLY | D3DUSAGE_DYNAMIC, D3DFVF_TEXTUREVERTEX2, D3DPOOL_DEFAULT, &m_pVertexBufferSuccessImage, nullptr);
	
		if (m_pVertexBufferFailureImage == nullptr)
			m_pApp->m_pd3dDevice->CreateVertexBuffer(sizeof(TEXTUREVERTEX2) * 3 * 6, D3DUSAGE_WRITEONLY | D3DUSAGE_DYNAMIC, D3DFVF_TEXTUREVERTEX2, D3DPOOL_DEFAULT, &m_pVertexBufferFailureImage, nullptr);
	
		return S_OK;
	}
	
	HRESULT Kia::CWndNewUpgrade::InvalidateDeviceObjects()
	{
		CWndBase::InvalidateDeviceObjects();
		SAFE_RELEASE(m_pVertexBufferGauge)
		SAFE_RELEASE(m_pVertexBufferSuccessImage)
		SAFE_RELEASE(m_pVertexBufferFailureImage)
		return S_OK;
	}
	
	HRESULT Kia::CWndNewUpgrade::DeleteDeviceObjects()
	{
		CWndBase::DeleteDeviceObjects();
		SAFE_RELEASE(m_pVertexBufferGauge)
		SAFE_RELEASE(m_pVertexBufferSuccessImage)
		SAFE_RELEASE(m_pVertexBufferFailureImage)
		return S_OK;
	}
	
	void Kia::CWndNewUpgrade::OnDraw(C2DRender* p2DRender)
	{
		if (result != 0)
		{
			constexpr int nExtensionPixel(1);
			static CRect rectStaticTemp;
	
			const WNDCTRL* const lpStatic = GetWndCtrl(WIDC_STATIC31);
			rectStaticTemp.TopLeft().y = lpStatic->rect.top;
			rectStaticTemp.TopLeft().x = lpStatic->rect.left + nExtensionPixel;
			rectStaticTemp.BottomRight().y = lpStatic->rect.bottom;
			rectStaticTemp.BottomRight().x = lpStatic->rect.right + nExtensionPixel;
			if (result == 1)
				m_pTheme->RenderGauge(p2DRender, &rectStaticTemp, 0xffffffff, m_pVertexBufferSuccessImage, m_pSuccessTexture);
			else
				m_pTheme->RenderGauge(p2DRender, &rectStaticTemp, 0xffffffff, m_pVertexBufferSuccessImage, m_pFailureTexture);
		}
	
		if (start)
		{
			constexpr int nExtensionPixel(1);
			static CRect rectStaticTemp;
	
			const WNDCTRL* const lpStatic = GetWndCtrl(WIDC_STATIC31);
			rectStaticTemp.TopLeft().y = lpStatic->rect.top;
			rectStaticTemp.TopLeft().x = lpStatic->rect.left + nExtensionPixel;
			rectStaticTemp.BottomRight().y = lpStatic->rect.bottom;
			int nGaugeWidth(lpStatic->rect.left + static_cast<int>(static_cast<float>(lpStatic->rect.right - lpStatic->rect.left) * m_fGaugeRate));
			nGaugeWidth = nGaugeWidth < lpStatic->rect.right ? nGaugeWidth : lpStatic->rect.right;
			rectStaticTemp.BottomRight().x = nGaugeWidth + nExtensionPixel;
			m_pTheme->RenderGauge(p2DRender, &rectStaticTemp, 0xffffffff, m_pVertexBufferGauge, m_pNowGaugeTexture);
		}
	
	
		LPWNDCTRL lpStatic = GetWndCtrl(WIDC_STATIC1);
		if (itemTex && lpStatic)
		{
			itemTex->Render(p2DRender, lpStatic->rect.TopLeft());
			if (lpStatic->rect.PtInRect(m_ptMouse))
			{
				CPoint pt2 = m_ptMouse;
				CRect rcCpy = lpStatic->rect;
				ClientToScreen(&pt2);
				ClientToScreen(&rcCpy);
				if (itemElem)
					g_WndMng.PutToolTip_Item(itemElem, pt2, &rcCpy);
			}
	
			if (start)
				RenderEnchant(p2DRender, lpStatic->rect.TopLeft() + CPoint(2, 0));
		}
	
		lpStatic = GetWndCtrl(WIDC_STATIC11);
		if (matTex && lpStatic)
		{
			matTex->Render(p2DRender, lpStatic->rect.TopLeft());
			if (matCount > 0)
				p2DRender->TextOut(lpStatic->rect.left, lpStatic->rect.bottom - 6, 0.8f, 0.8f, std::to_string(matCount).c_str(), 0xff000000);
		}
	
		lpStatic = GetWndCtrl(WIDC_STATIC41);
		if (scrollTex && lpStatic)
		{
			scrollTex->Render(p2DRender, lpStatic->rect.TopLeft());
			if (scrollCount > 0)
				p2DRender->TextOut(lpStatic->rect.left, lpStatic->rect.bottom - 6, 0.8f, 0.8f, std::to_string(scrollCount).c_str(), 0xff000000);
	
		}
		lpStatic = GetWndCtrl(WIDC_STATIC61);
		if (scroll2Tex && lpStatic)
		{
			scroll2Tex->Render(p2DRender, lpStatic->rect.TopLeft());
			if (scrollCount2 > 0)
				p2DRender->TextOut(lpStatic->rect.left, lpStatic->rect.bottom - 6, 0.8f, 0.8f, std::to_string(scrollCount2).c_str(), 0xff000000);
	
		}
	}
	BOOL Kia::CWndNewUpgrade::OnChildNotify(const UINT message, const UINT nID, LRESULT* pLResult)
	{
		switch (nID)
		{
			case WIDC_BUTTON_START:
				if (start || !itemElem)
					break;
	
				start = true;
				g_DPlay.PrimeSmelting();
	
				enchantStart = g_tmCurrent;
				enchantEnd = g_tmCurrent + GetUpgradeTimerT<EUpgradeTimer::Default>();
				successes = 0;
				failures = 0;
				break;
			case WIDC_BUTTON_STOP:
				if (!start)
					break;
				start = false;
				enchantStart = NULL_ID;
				enchantEnd = NULL_ID;
				break;
			case WIDC_BUTTON_MINUS:
				if (--maxValue == 0)
					maxValue = 1;
				maxLimitEdit->SetString(std::to_string(maxValue).c_str());
				break;
			case WIDC_BUTTON_PLUS:
				if (++maxValue > GetDefaultMax())
					maxValue = GetDefaultMax();
				maxLimitEdit->SetString(std::to_string(maxValue).c_str());
				break;
			case WIDC_BUTTON_RESET:
				if (itemElem)
				{
					itemElem->SetExtra(itemElem->GetExtra() - 1);
					itemElem = nullptr;
					itemTex = nullptr;
				}
				ResetMats();
				successes = 0;
				failures = 0;
				result = 0;
				break;
			default:
				break;
		}
		return CWndNeuz::OnChildNotify(message, nID, pLResult);
	}
	#endif
	```

> Note: Some of the defines might not be implemented depending on the version. This uses both old and new Ik3's, so the new one's can be removed. Some things might have been forgotten.


## Resource Edits
---
- **Resdata.inc**
    ```
    APP_SMELT_SAFETY "WndTile00.tga" "" 1 320 160 0x2410000 26
    {
    // Title String
    IDS_RESDATA_INC_006428
    }
    {
    // ToolTip
    IDS_RESDATA_INC_006429
    }
    {
        WTYPE_STATIC WIDC_STATIC1 "WndChgElemItem.bmp" 0 153 8 187 41 0x2220002 0 0 0 0 46 112 169
        {
        // Title String
        IDS_RESDATA_INC_006430
        }
        {
        // ToolTip
        IDS_RESDATA_INC_006431
        }
        WTYPE_STATIC WIDC_STATIC11 "WndChgElemItem.bmp" 0 8 60 40 92 0x2220002 0 0 0 0 46 112 169
        {
        // Title String
        IDS_RESDATA_INC_006432
        }
        {
        // ToolTip
        IDS_RESDATA_INC_006433
        }
        WTYPE_STATIC WIDC_STATIC41 "WndChgElemItem.bmp" 0 52 60 84 92 0x2220002 0 0 0 0 46 112 169
        {
        // Title String
        IDS_RESDATA_INC_006452
        }
        {
        // ToolTip
        IDS_RESDATA_INC_006453
        }
        WTYPE_STATIC WIDC_STATIC31 "Window02.bmp" 0 129 60 297 92 0x2220052 0 0 0 0 46 112 169
        {
        // Title String
        IDS_RESDATA_INC_006472
        }
        {
        // ToolTip
        IDS_RESDATA_INC_006473
        }
        WTYPE_BUTTON WIDC_BUTTON_START "ButtStart.bmp" 0 34 98 98 118 0x220010 0 0 0 0 46 112 169
        {
        // Title String
        IDS_RESDATA_INC_006492
        }
        {
        // ToolTip
        IDS_RESDATA_INC_006493
        }
        WTYPE_BUTTON WIDC_BUTTON_STOP "Buttstop.bmp" 0 154 98 218 118 0x220010 0 0 0 0 46 112 169
        {
        // Title String
        IDS_RESDATA_INC_006494
        }
        {
        // ToolTip
        IDS_RESDATA_INC_006495
        }
        WTYPE_STATIC WIDC_STATIC61 "WndChgElemItem.bmp" 0 88 60 120 92 0x2220002 0 0 0 0 46 112 169
        {
        // Title String
        IDS_RESDATA_INC_006496
        }
        {
        // ToolTip
        IDS_RESDATA_INC_006497
        }
        WTYPE_STATIC WIDC_TITLE_NOW_GRADE "" 0 8 8 142 26 0x2220010 0 0 0 0 46 112 169
        {
        // Title String
        IDS_RESDATA_INC_006524
        }
        {
        // ToolTip
        IDS_RESDATA_INC_006525
        }
        WTYPE_STATIC WIDC_TITLE_MAX_GRADE "" 0 202 8 304 25 0x2220010 0 0 0 0 46 112 169
        {
        // Title String
        IDS_RESDATA_INC_006526
        }
        {
        // ToolTip
        IDS_RESDATA_INC_006527
        }
        WTYPE_BUTTON WIDC_BUTTON_RESET "ButtReset.BMP" 0 226 98 298 118 0x220010 0 0 0 0 46 112 169
        {
        // Title String
        IDS_RESDATA_INC_006528
        }
        {
        // ToolTip
        IDS_RESDATA_INC_006529
        }
        WTYPE_EDITCTRL WIDC_EDIT_MAX_GRADE "WndEditTile00.tga" 1 242 32 266 50 0x20000 0 0 0 0 46 112 169
        {
        // Title String
        IDS_RESDATA_INC_006530
        }
        {
        // ToolTip
        IDS_RESDATA_INC_006531
        }
        WTYPE_STATIC WIDC_NOW_GRADE "" 0 67 28 87 46 0x2220010 0 0 0 0 46 112 169
        {
        // Title String
        IDS_RESDATA_INC_006532
        }
        {
        // ToolTip
        IDS_RESDATA_INC_006533
        }
        WTYPE_BUTTON WIDC_BUTTON_PLUS "ButItemPlus.BMP" 0 272 34 286 50 0x220010 0 0 0 0 46 112 169
        {
        // Title String
        IDS_RESDATA_INC_006534
        }
        {
        // ToolTip
        IDS_RESDATA_INC_006535
        }
        WTYPE_BUTTON WIDC_BUTTON_MINUS "ButItemMinus.BMP" 0 222 34 236 50 0x220010 0 0 0 0 46 112 169
        {
        // Title String
        IDS_RESDATA_INC_006536
        }
        {
        // ToolTip
        IDS_RESDATA_INC_006537
        }
    }
    ```
- **Character.inc**
    Find and remove all the other MMI_SMELT_SAFTEY besides the normal one.




## License
---
_FlyFF is an mmorpg created by GalaNet/Aeonsoft. This implementation was done by myself:_
| Account  | Site |
| -------- | ------- |
| Kiaemi. | Discord |
| Velvet | RageZone |
| M¿dScientist | Elitepvpers |

