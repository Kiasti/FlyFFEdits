# Anarchy
_Anarchy is a system that replaces lord. It allows players and staff to start world buffs. This feature was replicated from Clockworks flyff, so all the skills was reminiscent from one of their first versions. If there's a skill that is missing or you want, we might add it._
[![Anarchy](http://img.youtube.com/vi/FiifhvE5Iwc/0.jpg)](https://www.youtube.com/watch?v=FiifhvE5Iwc)  

---
## Features
- Users's can run a world based buff.

## Patch Notes
---
- [05/03/2021] Making the system more diverse. Rather than having a special effect that's reliant on the Anarchy Skill Id, now it's focusing on the DST in the resource. This lets you have multiple skills with similiar Anarchy specific effects. This also moves away from a hardcoded approach allowing you to set more. 
- [05/03/2021] Fixed stats not properly resetting or showing they have been applied.
- [05/03/2021] Fixed old compiler issue.
- [05/03/2021] Added tooltip to the Anarchy window.

## Resource Example
---
- 
    ```
    //id, Name, Cost(int64), timeLasting(1800000=30min), DST, ADJ, DST2, ADJ2, Description, icon, Stack
    	1 "Haste of the Powerful" 10 1800000 11 15 0 0 "Speed +15%" "Icon_Anarchy1.png" 0
    ```
    Anarchy has some special DST's you may use. You have to use the dst ID's rather than the name as well. Here are the following special dst's you may use.
    ```
    egg = 1000,
	dungeonCooldown = 1001,
	materialRate = 1002,
	pieceDrop = 1003,
	goldInc = 1004,
	collectingTime = 1005,
	experience = 1006,
    ```
    All of them are based on percentages. For instance, `1001 20 1002 5` would be `Dungeon Cooldown -20%, Material Rate + 5%`. You'll also have to add it to the description.
## Adding parts to the source
---
   - **InstanceDungeonBase.cpp**  
     _Have to include the anarchy file._
     ```CPP
     #ifdef __WORLDSERVER
     #ifdef __Anarchy
     #include "Anarchy.h"
     #endif
     #endif
     ```
     <br/>**CInstanceDungeonBase::TeleportToDungeon**  
     _When the user enters the dungeon, the cooldown is reduced based on the buff. This way, the cooldown is only reducde while the buff is active._
     ```CPP
        		if( pCT_Info->dwDungeonId != dwDungeonId )
        		{
        			DWORD dwRemainCoolTime = pCT_Info->dwCoolTime - GetTickCount();
        #ifdef __Anarchy
        			const unsigned long factor = AnarchyManage::getAnarchy().getFactors(anarchyDstType::dungeonCooldown);
        			dwRemainCoolTime -= dwRemainCoolTime * (static_cast<float>(factor) / 100.0f);
        #endif
     ``` 
     <br/>
   - **Mover.cpp**  
     _Have to include the anarchy file._
     ```CPP
     #ifdef __Anarchy
     #include "Anarchy.h"
     #endif
     ```
     <br/>**CMover::DropItem**  
     _If goldInc based skills Active._
     ```CPP
     #if __VER >= 9 // __EVENTLUA
        					nNumGold = (int)( nNumGold * prj.m_EventLua.GetGoldDropFactor() );
     #endif // __EVENTLUA
     #ifdef __Anarchy
        					const unsigned long factor = AnarchyManage::getAnarchy().getFactors(anarchyDstType::goldInc);
        					nNumGold *= 1 + static_cast<float>(factor) / 100.0f;
     #endif
     ``` 
     <br/> find: &nbsp;&nbsp;**CMover::ProcessPetExp(void)**  
     _Increase pet exp from buff_
     ```CPP
            if (pPet->GetLevel() != PL_EGG && pPet->GetLevel() != PL_S)
            {
            	if (pPet->GetExp() == MAX_PET_EXP)
            		return;
    
            	float fFactor = HasBuff(BUFF_ITEM, II_SYS_SYS_SCR_PET_TONIC_B) ? 1.5F : 1.0F;
        #ifdef __Anarchy
        		const unsigned long factor = AnarchyManage::getAnarchy().getFactors(anarchyDstType::egg);
        		fFactor += static_cast<float>(factor) / 100.0f;
        #endif
     ```
     <br/> find: &nbsp;&nbsp;**CMover::StartCollecting( void )**  
     _Increase Collecting speed_
        ```CPP
        #ifdef __CLIENT
        	CItemElem*			 pCollector = GetCollector();
        	CCollectingProperty* pProperty	= CCollectingProperty::GetInstance();
        
        #ifdef __Anarchy
        	if (pCollector)
        	{
        		m_nMaxCltTime = pProperty->GetCool(pCollector->GetAbilityOption()) + 1;
        		const float temp = static_cast<float>(AnarchyManage::getAnarchy().getFactors(anarchyDstType::collectingTime)) / 100.0f;
        		m_nMaxCltTime -= temp >= 0.99 ? m_nMaxCltTime * 0.99f : m_nMaxCltTime * temp;
        	}
        #else
        	if( pCollector ) m_nMaxCltTime	= pProperty->GetCool( pCollector->GetAbilityOption() ) + 1;
        #endif
        ```
      <br/>
   - **MoverParam.cpp**  
       _Include the file_  
       ```CPP
        #ifdef __Anarchy
        #include "Anarchy.h"
        #endif
        ```
       <br/>find:&nbsp;**CMover::GetPieceItemDropRateFactor**
       ```CPP
        #ifdef __Anarchy
        	const unsigned long factor = AnarchyManage::getAnarchy().getFactors(anarchyDstType::pieceDrop);
        	fFactor += static_cast<float>(factor) / 100.0f;
    	#endif
        #endif // __WORLDSERVER
        	return fFactor;
       ```
       <br/>find: &nbsp;&nbsp;**CMover::GetExpFactor**
       ```CPP
       #ifdef __Anarchy
        	const unsigned long factor = AnarchyManage::getAnarchy().getFactors(anarchyDstType::experience);
        	fFactor += static_cast<float>(factor) / 100.0f;
        #endif
        #if __VER >= 12 // __LORD
        	ILordEvent* pEvent		= CSLord::Instance()->GetEvent();
        	fFactor		*= pEvent->GetEFactor();
       ```      
   - **Project.cpp**  
       find: &nbsp;&nbsp;**CDropItemGenerator::GetAt**
       ```CPP
        	if( fProbability > 0.0f ) 
        	{
        		DROPITEM* lpDropItem	= &m_dropItems[ nIndex ];
        		DWORD dwProbability		= lpDropItem->dwProbability;
        		DWORD dwRand = xRandom(3000000000);
        
        #ifdef __Anarchy
        		ItemProp* iProp = prj.GetItemProp(lpDropItem->dwIndex);
        		if (iProp->dwItemKind2 == IK2_MATERIAL)
        		{
        			const unsigned long factor = AnarchyManage::getAnarchy().getFactors(anarchyDstType::materialRate);
        			dwRand = static_cast<unsigned long>(static_cast<float>(dwRand) / static_cast<float>(factor) / 100.0f);
        		}
        #endif
       ```
       <br/>find: &nbsp;&nbsp;**CProject::OpenProject**        
       ```CPP
         #ifdef __Anarchy
        	AnarchyManage::loadAnarchySkillFile();
        #endif
	       return TRUE;
       ```
   - **WndManager.h**
        ```CPP
        #ifdef __Anarchy
        #include "Anarchy.h"
        #endif
        ```
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Add Bottom under public: in wndmng 
        ```CPP
        #ifdef __Anarchy
        	AnarchyWnd* pWndAnarchy;
        #endif
        ```
   - **WndManager.cpp**
    &nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;find &nbsp;&nbsp;&nbsp; **CWndMgr::CWndMgr()**
        ```CPP
        #ifdef __Anarchy
	          pWndAnarchy = nullptr;
        #endif
        ```
        &nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;find &nbsp;&nbsp;&nbsp; **CWndMgr::Free()**
        ```CPP
        #ifdef __Anarchy
	        SAFE_DELETE(pWndAnarchy);
        #endif
        ```
        &nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;find &nbsp;&nbsp;&nbsp; **CWndMgr::OnDestoryChildWnd**
        ```CPP
        #ifdef __Anarchy
        // If this errors, remove the "else"
	       else if (pWndAnarchy == pWndChild)
        	{
        		SAFE_DELETE(pWndAnarchy);
        		pWndChild = nullptr;
        	}
        #endif
        ```
   - **WndWorld.cpp**
    &nbsp;&nbsp;&nbsp;&nbsp; find &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _BOOL CWndWorld::OnCommand(UINT nID, DWORD dwMessage, CWndBase* pWndBase)_
        ```CPP
        		switch (nID)
        		{
        #ifdef __Anarchy
        			case MMI_Anarchy:
        				SAFE_DELETE(g_WndMng.pWndAnarchy);
        				g_WndMng.pWndAnarchy = new AnarchyWnd;
        				g_WndMng.pWndAnarchy->Initialize();
        				break;
        #endif
        ```
        
       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; add and replace
        ```CPP
        #ifdef __Anarchy
        #include "Anarchy.h"
        namespace std
        {
        	inline std::string to_string(const std::string& s) { return s; }
        	inline std::string to_string(const char* s) { return s; }
        }
        
        namespace kiaUtil
        {
        	template<typename ... Args> static std::string build(Args && ... args)
        	{
        		std::string str;
        		char wew[]{ 0, ((void)(str += std::to_string(args)), 0) ... };
        		static_cast<void>(wew);
        		return str;
        	}
        }
        extern BOOL IsDst_Rate(int nDstParam);
        extern char *FindDstString(int nDstParam);
        #endif
        #if __VER >= 12 // __LORD
        #define	TTI_LORD_EVENT	123456789
        void CWndWorld::RenderEventIcon( C2DRender* p2DRender, BUFFICON_INFO* pInfo, CPoint ptMouse )
        {
        	RECT rectHittest;
        #ifdef __Anarchy
        	Anarchy& inst = AnarchyManage::getAnarchy();
        	for (std::vector<AnarchySkillOn>::const_iterator it = inst.runningSkills.cbegin(); it !=            inst.runningSkills.cend();)
        	{
        		if (g_tmCurrent > it->tickEnd)
        		{
        			it = inst.runningSkills.erase(it);
        			continue;
        		}
        
        		pInfo->pt.x += (32 + 5);
        		const AnarchySkillParameter& skillParam = inst.findSkillInfo(it->skillId);
        		if (skillParam.skillId == NULL_ID)
        			continue;
        
        		CTexture* pTexture = m_textureMng.AddTexture(D3DDEVICE, MakePath(DIR_ICON,    skillParam.icon.c_str()),      0xffff00ff);
        		if (pTexture)
        			p2DRender->RenderTexture(pInfo->pt, pTexture, 192);
        
        		SetRect(&rectHittest, pInfo->pt.x, pInfo->pt.y, pInfo->pt.x + 32, pInfo->pt.y + 32);
        		ClientToScreen(&rectHittest);
        
        		CEditString editString;
        		editString.AddString(skillParam.name.c_str(), D3DCOLOR_XRGB(0, 93, 0), ESSTY_BOLD);
        		
        		const long long buffTimer = (it->tickEnd - g_tmCurrent) / 1000;
        		CTimeSpan timeSpan(0, 0, 0, buffTimer);
        		std::string tmp = kiaUtil::build("\nTime Remaining: ", timeSpan.GetHours(), "h, ",    timeSpan.GetMinutes(),      "m, ", timeSpan.GetSeconds(), "s");
        		editString.AddString(tmp.c_str(), 0xff000000);
        
        		tmp = kiaUtil::build("\n", skillParam.description);
        		editString.AddString(tmp.c_str(), 0xFF323232);
        		g_toolTip.PutToolTip(TTI_LORD_EVENT, editString, rectHittest, ptMouse, 1);
        		
        		++pInfo->nCount;
        		RenderOptBuffTime(p2DRender, pInfo->pt, timeSpan, D3DCOLOR_XRGB(240, 240, 0));
        		if ((pInfo->nCount % m_nLimitBuffCount) == 0)
        		{
        			pInfo->pt.x = (m_rectWindow.Width() / 2) + 75;
        			pInfo->pt.y += GetBuffTimeGap();
        		}
        		++it;
        	}
        
        #else
        	ILordEvent* pEvent	= CCLord::Instance()->GetEvent();
        	if (pEvent)
        	{
        		for (int i = 0; i < pEvent->GetComponentSize(); i++)
        		{
        			CLEComponent* pComponent = pEvent->GetComponentAt(i);
        			pInfo->pt.x += (32 + 5);
        			p2DRender->RenderTexture(pInfo->pt, pComponent->GetTexture(), 192);
        			SetRect(&rectHittest, pInfo->pt.x, pInfo->pt.y, pInfo->pt.x + 32, pInfo->pt.y + 32);
        			ClientToScreen(&rectHittest);
        			// ±ºÁÖ %s´ÔÀÌ °æÇèÄ¡ %3.1f%%, µå·Ó·ü %3.1f%% »ó½Â ÀÌº¥Æ®¸¦ ÁøÇà Áß ÀÔ´Ï´Ù."
        			CEditString editString;
        			char szTooltip[255] = { 0, };
        			sprintf(szTooltip, prj.GetText(TID_GAME_LORD_EVENT_TOOLTIP),
        				CPlayerDataCenter::GetInstance()->GetPlayerString(pComponent->GetIdPlayer()),
        				pComponent->GetEFactor() * 100, pComponent->GetIFactor() * 100);
        			editString.AddString(szTooltip, D3DCOLOR_XRGB(0, 93, 0), ESSTY_BOLD);
        			CString strRest;
        			CTimeSpan timeSpan(0, 0, pComponent->GetTick(), 0);
        			strRest.Format("\n%d", timeSpan.GetTotalMinutes());
        			editString += strRest;
        			g_toolTip.PutToolTip(TTI_LORD_EVENT, editString, rectHittest, ptMouse, 1);
        			pInfo->nCount++;
        			RenderOptBuffTime(p2DRender, pInfo->pt, timeSpan, D3DCOLOR_XRGB(240, 240, 0));
        			if ((pInfo->nCount % m_nLimitBuffCount) == 0)
        			{
        				pInfo->pt.x = (m_rectWindow.Width() / 2) + 75;
        				pInfo->pt.y += GetBuffTimeGap();
        			}
        		}
        	}
        #endif
        }
        #endif	// __LORD
        ```
   -  **MsgHdr.h**
        ```CPP
        #ifdef __Anarchy
        #define	PACKETTYPE_AnarchySkills static_cast<unsigned long>(0x9FAF9F00)
        #define PACKETTYPE_AnarchySkillsUse static_cast<unsigned long>(0x9FAF9F01)
        #define SNAPSHOTTYPE_AnarchySkillUse static_cast<unsigned short>(0x9F00)
        #define SNAPSHOTTYPE_AnarchySkills static_cast<unsigned short>(0x9F01)
        #endif
      ```
   -  **Databaseserver.cpp**
        ```CPP
        #ifdef __Anarchy
        #include "Anarchy.h"
        #endif
        ```
        ```CPP
        	CTLord::Instance()->CreateColleagues();
        #ifdef __Anarchy
        	AnarchyManage::getDb().restoreRunningSkills();
        #endif
        ```
   -  **dpTrans.cpp**
        ```CPP
        #ifdef __Anarchy
        #include "Anarchy.h"
        #endif
        ```
        ```CPP
        #ifdef __Anarchy
        	ON_MSG(PACKETTYPE_AnarchySkillsUse, &CDPTrans::onAnarchySkillUse);
        #endif
        ```
        in : _CDPTrans::SysMessageHandler_
        ```CPP
        #if __VER >= 12 // __LORD
				// ¿ùµå ¼­¹ö¿¡ ±ºÁÖ Á¤º¸ Àü¼Û
				CTLord::Instance()->PostRequest( CTLord::eInit, NULL, 0, lpCreatePlayer->dpId );
        #endif	// __LORD
        #ifdef __Anarchy
				AnarchyManage::getDb().PostRequest(AnarchyDBController::eInit, nullptr, 0, lpCreatePlayer->dpId);
        #endif
        ```
        Add to bottom
        ```CPP
        #ifdef __Anarchy
        void CDPTrans::onAnarchySkillUse(CAr & ar, DPID dpid, DPID dpidCache, DPID dpidUser, LPBYTE lpBuf, u_long         uBufSize)
        {
        	AnarchyManage::getDb().postMessage(AnarchyDBController::eAnarchySkillUse, lpBuf, uBufSize);
        }
        void CDPTrans::sendAnarchy(DPID dpid)
        {
        	BEFORESENDDUAL(ar, PACKETTYPE_AnarchySkills, DPID_UNKNOWN, DPID_UNKNOWN);
        	AnarchyManage::getAnarchy().serializeRunningSkills(ar);
        	SEND(ar, this, dpid);
        }
        
        #endif
        ```
   - **dptrans.h**
        ```CPP
         #ifdef __Anarchy
        	void onAnarchySkillUse(CAr & ar, DPID dpid, DPID dpidCache, DPID dpidUser, LPBYTE lpBuf, u_long uBufSize);
        	void sendAnarchy(DPID dpid);
        #endif
	   ```
   - **Project.cpp** _(DatabaseServer)_
        ```CPP
        #ifdef __Anarchy
        #include "Anarchy.h"
        #endif
        ```
        ```CPP
        #ifdef __GUILD_HOUSE_MIDDLE
        	GuildHouseMng->LoadScript();
        #endif // __GUILD_HOUSE_MIDDLE
        #ifdef __Anarchy
        	AnarchyManage::loadAnarchySkillFile();
        #endif
        ```
      
   - **dpClient.cpp** _(Neuz)_
      ```CPP
        // In the list of snapshots
        #ifdef __Anarchy
        			case SNAPSHOTTYPE_AnarchySkillUse: onAnarchySkillUse(ar); break;
        			case SNAPSHOTTYPE_AnarchySkills: onAnarchySkills(ar); break;
        #endif
        
        // Add to bottom
        #ifdef __Anarchy
        void CDPClient::sendAnarchySkill(const unsigned long skillId)
        {
        	BEFORESENDSOLE(ar, PACKETTYPE_AnarchySkillsUse, DPID_UNKNOWN);
        	ar << skillId;
        	SEND(ar, this, DPID_SERVERPLAYER);
        }
        #include "Anarchy.h"
        void CDPClient::onAnarchySkillUse(CAr& ar)
        {
        	unsigned long skillId;
        	ar >> skillId;
        
        	Anarchy& inst = AnarchyManage::getAnarchy();
        	const AnarchySkillParameter& skillP = inst.findSkillInfo(skillId);
        	if (!skillP.isEmpty())
        	{
        		for (std::vector<AnarchySkillOn>::iterator it = inst.runningSkills.begin(); it !=         inst.runningSkills.end(); ++it)
        		{
        			if (it->skillId == skillId)
        			{
        				it->tickEnd += skillP.timeLasting;
        				return;
        			}
        		}
        		inst.runningSkills.emplace_back(skillId, g_tmCurrent + skillP.timeLasting);
        
        		if (g_pPlayer)
        		{
        			for (int i = 0; i < 2; ++i)
        			{
        				if (Anarchy::isAnarchyDst(static_cast<anarchyDstType>(skillP.bonus[i].dst)))
        				{
        					if (skillP.bonus[i].dst == static_cast<unsigned short>(anarchyDstType::collectingTime))
        					{
        						const float temp = static_cast<float>(skillP.bonus[i].adj) / 100.0f;
        						g_pPlayer->m_nMaxCltTime -= temp >= 0.99 ? static_cast<float>(g_pPlayer->m_nMaxCltTime) * 0.99f : static_cast<float>(g_pPlayer->m_nMaxCltTime) * temp;
        						if (g_pPlayer->m_nMaxCltTime < GetTickCount())
        							g_pPlayer->m_nMaxCltTime = GetTickCount();
        					}
        				}
        			}
        		}
        	}
        }
        void CDPClient::onAnarchySkills(CAr& ar)
        {
        	Anarchy& inst = AnarchyManage::getAnarchy();
        	inst.serializeRunningSkills(ar);
        }
        #endif
        ```
   -  **DpClient.h**  _(neuz)_
        ```CPP
        #ifdef __Anarchy
        	void sendAnarchySkill(const unsigned long skillId);
        	void onAnarchySkillUse(CAr &ar);
        	void onAnarchySkills(CAr &ar);
        #endif
        ```
   -  **DpDatabaseClient.cpp** _(Worldserver)_
        ```CPP
        // Where all the other ON_MSG are
        #ifdef __Anarchy
    	ON_MSG(PACKETTYPE_AnarchySkills, &CDPDatabaseClient::onAnarchy);
        #endif
        ```
        in **CDpDatabaseClient::OnJoin**
        ```CPP
        #ifdef __Anarchy
	        	pUser->sendAnarchySkills();
        #endif

        #ifdef __ON_ERROR
        		}
        		catch( ... )
		```
		Add to bottom of file
		```CPP
        #ifdef __Anarchy
        #include "Anarchy.h"
        void CDPDatabaseClient::sendAnarchyUse(const unsigned long idPlayer, const unsigned long skillId)
        {
        	BEFORESENDDUAL(ar, PACKETTYPE_AnarchySkillsUse, DPID_UNKNOWN, DPID_UNKNOWN);
        	ar << skillId;
        	SEND(ar, this, DPID_SERVERPLAYER);
        }
        
        void CDPDatabaseClient::onAnarchy(CAr & ar, DPID, DPID)
        {
        	AnarchyManage::getAnarchy().serializeRunningSkills(ar);
        }
        #endif
        ```
   -  **DpDatabaseClient.h**
        ```CPP
         #ifdef __Anarchy
    	void sendAnarchyUse(const unsigned long idPlayer, const unsigned long skillId);
    	void onAnarchy(CAr & ar, DPID, DPID);
        #endif
        ```
   -  **DPSrvr.cpp**
        ```CPP
        #ifdef __Anarchy
        ON_MSG(PACKETTYPE_AnarchySkillsUse, &CDPSrvr::onAnarchyUse);
        #endif
        ```
        ```CPP
        #ifdef __Anarchy
        #include "Anarchy.h"
        void CDPSrvr::onAnarchyUse(CAr& ar, DPID dpidCache, DPID dpidUser, LPBYTE lpBuf, u_long uBufSize)
        {
        	CUser* pUser = g_UserMng.GetUser(dpidCache, dpidUser);
        	if (IsValidObj(pUser))
        	{
        		unsigned long skillId;
        		ar >> skillId;
        
        		Anarchy& inst = AnarchyManage::getAnarchy();
        		const AnarchySkillParameter& skillP = inst.findSkillInfo(skillId);
        
        		if (!skillP.isEmpty())
        		{
        			if (pUser->GetTotalGold() < skillP.cost)
        			{
        				pUser->AddText("You dont have enough money");
        				return;
        			}
        
        			bool inVec = false;
        			for (std::vector<AnarchySkillOn>::iterator it = inst.runningSkills.begin(); it !=         inst.runningSkills.end(); ++it)
        			{
        				if (it->skillId == skillId)
        				{
        					if (!skillP.isStackable())
        					{
        						pUser->AddText("This skill is not Stackable");
        						return;
        					}
        					it->tickEnd += skillP.timeLasting;
        					inVec = true;
        				}
        			}
        
        			pUser->RemoveTotalGold(skillP.cost);
        			g_dpDBClient.sendAnarchyUse(pUser->m_idPlayer, skillId);
        
        			if (!skillP.isStackable())
        				inst.runningSkills.push_back({ skillId, g_tmCurrent + skillP.timeLasting });
        
        			g_UserMng.userIterator([skillId, &skillP, inVec](const std::map<DWORD, CUser*>::iterator it)
        				{
        					CUser* nUser = it->second;
        					if (IsValidObj(nUser))
        					{
        						if (!inVec)
        						{
        							for (int i = 0; i < 2; ++i)
        								if (skillP.bonus[i].dst != 0 && !Anarchy::isAnarchyDst(static_cast<anarchyDstType>(skillP.bonus[i].dst)))
        									nUser->SetDestParam(skillP.bonus[i].dst, skillP.bonus[i].adj, NULL_CHGPARAM, true);
        						}
        						nUser->sendAnarchySkill(skillId);
        					}
        				});
        			std::string message = pUser->GetName();
        			message += " has used ";
        			message += skillP.name;
        			g_DPCoreClient.SendSystem(message.c_str());
        		}
        	}
        }
        #endif
        ```
   -  **dpsrvr.h**
        ```CPP
        #ifdef __Anarchy
	    void onAnarchyUse(CAr &ar, DPID dpidCache, DPID dpidUser, LPBYTE lpBuf, u_long uBufSize);
        #endif
        ```
   -  **ThreadMng.cpp**
        ```CPP
        #ifdef __Anarchy
        #include "Anarchy.h"
        #endif
        ```
        find and add
        ```CPP
        #ifdef __Anarchy
        			AnarchyManage::getAnarchy().processAnarchy();
        #endif

        #if __VER >= 14 // __INSTANCE_DUNGEON
        			{
        				CInstanceDungeonParty::GetInstance()->Process();
        			}
        #endif // __INSTANCE_DUNGEON
        ```
   -  **User.cpp** _(Worldserver)_
        ```CPP
        #ifdef __Anarchy
        #include "Anarchy.h"
        #endif
        ```
        in _CUser::ProcessCollecting(void)_
        ```CPP
        #ifdef __Anarchy
        		int collectingSpeed = pProperty->GetCool(pCol->GetAbilityOption());
        		const float tempf = static_cast<float>(AnarchyManage::getAnarchy().getFactors(anarchyDstType::collectingTime)) / 100.0f;
        		collectingSpeed -= tempf < 0.99f ? collectingSpeed * tempf : collectingSpeed * 0.99f;
        		if (++m_nCollecting >= collectingSpeed)
        #else
        		if (++m_nCollecting >= pProperty->GetCool(pCol->GetAbilityOption()))
        #endif
        ```
        add to bottom of file
        ```CPP
        #ifdef __Anarchy
        void CUser::sendAnarchySkill(const unsigned long skillId)
        {
        	if (IsDelete()) return;
        
        	m_Snapshot.cb++;
        	m_Snapshot.ar << GetId() << SNAPSHOTTYPE_AnarchySkillUse << skillId;
        }
        
        void CUser::sendAnarchySkills()
        {
        	if (IsDelete())
        		return;
        
        	Anarchy& tmpInst = AnarchyManage::getAnarchy();
        	if (tmpInst.runningSkills.empty())
        		return;
        
        	m_Snapshot.cb++;
        	m_Snapshot.ar << GetId();
        	m_Snapshot.ar << SNAPSHOTTYPE_AnarchySkills;
        	tmpInst.serializeRunningSkills(m_Snapshot.ar);
        
        	for (std::vector<AnarchySkillOn>::iterator it = tmpInst.runningSkills.begin(); it !=         tmpInst.runningSkills.end(); ++it)
        	{
        		const AnarchySkillParameter& skillP = tmpInst.findSkillInfo(it->skillId);
        		if (skillP.isEmpty())
        			continue;
        
        		for (int i = 0; i < 2; ++i)
        			if (skillP.bonus[i].dst != 0 && !Anarchy::isAnarchyDst(static_cast<anarchyDstType>(skillP.bonus[i].dst)))
        				SetDestParam(skillP.bonus[i].dst, skillP.bonus[i].adj, NULL_CHGPARAM, true);
        	}
        }
        
        void CUserMng::userIterator(std::function<void(const std::map<DWORD, CUser*>::iterator it)>&& func)
        {
        	for (std::map<DWORD, CUser*>::iterator it = m_users.begin(); it != m_users.end(); ++it)
        		func(it);
        }
        #endif
        ```
   -  **user.h**
        In class _CUser_ under public scope
        ```CPP
        #ifdef __Anarchy
        	void sendAnarchySkill(const unsigned long skillId);
	        void sendAnarchySkills();
        #endif
        ```
        in class _CUserMng_ under public scope
        ```
        #ifdef __Anarchy
    	    void userIterator(std::function<void(std::map<DWORD, CUser*>::iterator it)>&& func);
        #endif
        ```
   -  Make a file named **Anarchy.h** and **Anarchy.cpp** and add the following to it. Afterwards, add the file to the neuz project, Database, and Worldserver.

        **Anarchy.h**
        ```CPP
        #pragma once
        
        #ifdef __Anarchy
        #include <vector>
        #include <string>
        #include <functional>
        #ifdef __DBSERVER
        #include "../_database/dbmanager.h"
        #endif
        
        enum class anarchyDstType : unsigned long
        {
        	egg = 1000,
        	dungeonCooldown = 1001,
        	materialRate = 1002,
        	pieceDrop = 1003,
        	goldInc = 1004,
        	collectingTime = 1005,
        	experience = 1006,
        	maxDst = 7
        };
        
        struct AnarchySkillBonus
        {
        	unsigned short dst;
        	long adj;
        	explicit AnarchySkillBonus() : dst(0), adj(0) { }
        	AnarchySkillBonus(const unsigned short a, const long b) :dst(a), adj(b) {}
        };
        
        struct AnarchySkillParameter
        {
        	unsigned long skillId;
        	long long cost;
        	long long timeLasting;
        	AnarchySkillBonus bonus[2]{};
        	std::string icon;
        	std::string name;
        	std::string description;
        	bool stackable;
        
        	[[nodiscard]] bool isStackable() const { return stackable; }
        	[[nodiscard]] bool isEmpty() const { return skillId == NULL_ID; }
        	explicit AnarchySkillParameter(): skillId(NULL_ID), cost(0), timeLasting(0), stackable(false)
        	{
        	}
        };
        
        struct AnarchySkillOn 
        {
        	unsigned long skillId;
        	long long tickEnd;
        	explicit AnarchySkillOn() : skillId(0), tickEnd(0) { }
        	AnarchySkillOn(const unsigned long a, const long long b) : skillId(a), tickEnd(b) {}
        };
        
        class Anarchy
        {
        public:
        #if _HAS_CXX17
        	inline static AnarchySkillParameter empty{};
        #else
        	static AnarchySkillParameter empty;
        #endif
        	static bool isAnarchyDst(anarchyDstType&& dst);
        	std::vector<AnarchySkillParameter> anarchyList;
        	std::vector<AnarchySkillOn> runningSkills;
        
        	[[nodiscard]] unsigned long getFactors(anarchyDstType&& type);
        	[[nodiscard]] bool findRunningSkill(unsigned long id) const;
        	const AnarchySkillParameter& findSkillInfo(unsigned long skill);
        	void serializeRunningSkills(CAr &ar);
        	void iterator(std::function<void(std::vector<AnarchySkillParameter>::const_iterator it)> && func);
        	[[nodiscard]] size_t listSize() const;
        
        #ifndef __DBSERVER
        	void processAnarchy();
        #endif
        #ifdef __DBSERVER
        	AnarchySkillOn& lastAddedSkill();
        #endif
        };
        
        #ifdef __DBSERVER
        #include "../DatabaseServer/dbcontroller.h"
        #include "dptrans.h"
        extern AppInfo g_appInfo;
        
        class AnarchyDBController : public CDbController
        {
        	void OnTimer() override;
        	bool addSkill(const unsigned long skillId);
        	void Handler(const LPDB_OVERLAPPED_PLUS pov, const DWORD dwCompletionKey) override;
        public:
        	enum { eInit, eAnarchySkillUse };
        	void restoreRunningSkills();
        	bool postMessage(int nQuery, BYTE* lpBuf = nullptr, int nBufSize = 0, DWORD dwCompletionKey = 0U);
        };
        #endif
        
        class AnarchyManage
        {
        	static Anarchy ana;
        #ifdef __DBSERVER
        	static AnarchyDBController controller;
        #endif
        
        public:
        	static Anarchy& getAnarchy() { return ana; }
        #ifdef __DBSERVER
        	static AnarchyDBController& getDb() { return controller; }
        	static void initDb();
        #endif
        	static bool loadAnarchySkillFile();
        };
        
        #ifdef __CLIENT
        class AnarchyWnd final : public CWndNeuz
        {
        	int page;
        	int maxPage;
        
        	CWndButton* useButtons[4];
        	CWndButton* pageButtons[2];
        	CWndStatic* skillImages[4];
        
        public:
        	AnarchyWnd();
        	~AnarchyWnd();
        
        	BOOL Initialize(CWndBase* pWndParent = NULL, DWORD nType = MB_OK) override;
        	BOOL OnChildNotify(UINT message, UINT nID, LRESULT* pLResult) override;
        	void OnDraw(C2DRender* p2DRender) override;
        	void OnInitialUpdate() override;
        };
        #endif
        #endif
        ```
   -  **Dbcontroller.h** (in class CDbController)
        ```CPP
        	[[nodiscard]] void* getIocp() const { return m_hIocp; }
        ```		
   -  **Anarchy.cpp**
        ```CPP
        #include "StdAfx.h"
        #include "Anarchy.h"
        
        #ifdef __Anarchy
        #if !_HAS_CXX17
        AnarchySkillParameter Anarchy::empty = AnarchySkillParameter();
        #endif
        
        Anarchy AnarchyManage::ana;
        #ifdef __DBSERVER
        AnarchyDBController AnarchyManage::controller;
        #endif
        
        bool AnarchyManage::loadAnarchySkillFile()
        {
        	CScanner s;
        	if (!s.Load("Anarchy.txt"))
        		return false;
        
        	ana.anarchyList.clear();
        
        	s.GetToken();
        	while (s.tok != FINISHED)
        	{
        		if (s.Token == _T("AnarchySkills"))
        		{
        			s.GetToken();
        			unsigned long id = s.GetNumber();
        			while (*s.token != '}')
        			{
        				AnarchySkillParameter pSkill;
        				pSkill.skillId = id;
        				s.GetToken();
        				pSkill.name = s.Token;
        				pSkill.cost = s.GetInt64();
        				pSkill.timeLasting = s.GetNumber();
        				pSkill.bonus[0].dst = s.GetNumber();
        				pSkill.bonus[0].adj = s.GetNumber();
        				pSkill.bonus[1].dst = s.GetNumber();
        				pSkill.bonus[1].adj = s.GetNumber();
        				s.GetToken();
        				pSkill.description = s.Token;
        				s.GetToken();
        				pSkill.icon = s.Token;
        				pSkill.stackable = (s.GetNumber() != 0) ? true : false;
        
        				ana.anarchyList.push_back(std::move(pSkill));
        				id = s.GetNumber();
        			}
        		}
        		else
        		{
        			break;
        		}
        	}
        	ana.runningSkills.reserve(ana.anarchyList.size());
        #ifdef __DBSERVER
        	initDb();
        #endif
        	return true;
        
        }
        
        #ifdef __DBSERVER
        void AnarchyManage::initDb()
        {
        	controller.CreateDbHandler(MIN(1));
        }
        #endif
        
        bool Anarchy::isAnarchyDst(anarchyDstType&& dst)
        {
        	switch (dst)
        	{
        		case anarchyDstType::egg:
        		case anarchyDstType::dungeonCooldown:
        		case anarchyDstType::materialRate:
        		case anarchyDstType::pieceDrop:
        		case anarchyDstType::goldInc:
        		case anarchyDstType::collectingTime:
        		case anarchyDstType::experience:
        			return true;
        		default:
        			return false;
        	}
        }
        
        size_t Anarchy::listSize() const
        {
        	return anarchyList.size();
        }
        
        bool Anarchy::findRunningSkill(const unsigned long id) const
        {
        	return std::find_if(runningSkills.cbegin(), runningSkills.cend(), [&id](const AnarchySkillOn& onSkill) -> bool        {
        		return onSkill.skillId == id; }) != runningSkills.end();
        }
        
        unsigned long Anarchy::getFactors(anarchyDstType&& type)
        {
        	unsigned long factor = 0;
        	for (std::vector<AnarchySkillOn>::iterator it = runningSkills.begin(); it != runningSkills.end(); ++it)
        	{
        		const AnarchySkillParameter& skillP = findSkillInfo(it->skillId);
        		if (!skillP.isEmpty())
        		{
        			for (int k = 0; k < 2; ++k)
        			{
        				if (skillP.bonus[k].dst == static_cast<unsigned short>(type))
        					factor += skillP.bonus[k].adj;
        			}
        		}
        	}
        	return factor;
        }
        
        
        const AnarchySkillParameter& Anarchy::findSkillInfo(const unsigned long skill)
        {
        	const std::vector<AnarchySkillParameter>::const_iterator it = std::find_if(anarchyList.begin(),         anarchyList.end(), [&skill](const AnarchySkillParameter& param) -> bool
        		{
        			return skill == param.skillId;
        		});
        	if (it != anarchyList.end())
        		return *it;
        
        	return empty;
        }
        
        void Anarchy::serializeRunningSkills(CAr &ar)
        {
        	if (ar.IsStoring())
        	{
        		ar << static_cast<unsigned long>(runningSkills.size()); 
        		for (std::vector<AnarchySkillOn>::const_iterator it = runningSkills.begin(); it != runningSkills.end();        ++it)
        		{
        			ar << it->skillId;
        #ifndef __DBSERVER
        			ar << it->tickEnd - g_tmCurrent;
        #else
        			ar << it->tickEnd;
        #endif
        		}
        	}
        	else
        	{
        		unsigned long size;
        		ar >> size;
        		unsigned long skillId;
        		long long tickEnd;
        
        #ifdef __CLIENT
        		runningSkills.clear();
        #endif
        
        		for (unsigned long i = 0; i < size; ++i)
        		{
        			ar >> skillId >> tickEnd;
        #ifndef __DBSERVER
        			runningSkills.push_back({ skillId, g_tmCurrent + tickEnd });
        #else
        			runningSkills.push_back({ skillId, tickEnd });
        #endif
        		}
        	}
        }
        
        
        
        void Anarchy::iterator(std::function<void(const std::vector<AnarchySkillParameter>::const_iterator it)> && func)
        {
        	for (std::vector<AnarchySkillParameter>::const_iterator it = anarchyList.begin(); it != anarchyList.end();         ++it)
        		func(it);
        }
        
        #ifdef __WORLDSERVER
        #include "user.h"
        extern CUserMng g_UserMng;
        void Anarchy::processAnarchy()
        {
        	for (std::vector<AnarchySkillOn>::iterator it = runningSkills.begin(); it != runningSkills.end(); )
        	{
        		if (g_tmCurrent > it->tickEnd)
        		{
        			const unsigned long skillId = it->skillId;
        			const AnarchySkillParameter& skillP = AnarchyManage::getAnarchy().findSkillInfo(skillId);
        			if (!skillP.isEmpty())
        			{
        				g_UserMng.userIterator([skillId, &skillP](const std::map<DWORD, CUser*>::iterator it)
        					{
        						CUser* nUser = it->second;
        						if (IsValidObj(nUser))
        						{
        							for (int i = 0; i < 2; ++i)
        								if (skillP.bonus[i].dst != 0 &&         !Anarchy::isAnarchyDst(static_cast<anarchyDstType>(skillP.bonus[i].dst)))
        									nUser->ResetDestParam(skillP.bonus[i].dst, skillP.bonus[i].adj, true);
        						}
        					});
        			}
        			it = runningSkills.erase(it);
        		}
        		else
        			++it;
        	}
        }
        #endif
        
        
        #ifdef __DBSERVER
        AnarchySkillOn& Anarchy::lastAddedSkill(){
        	return runningSkills[runningSkills.size() - 1];
        }
        
        
        void AnarchyDBController::OnTimer()
        {
        	Anarchy& instance = AnarchyManage::getAnarchy();
        	for (std::vector<AnarchySkillOn>::iterator it = instance.runningSkills.begin(); it !=         instance.runningSkills.end(); )
        	{
        		if (--it->tickEnd < 0)
        		{
        			GetQueryObject()->Execute("AnarchyRemoveSkill %d, %d", g_appInfo.dwSys, it->skillId);
        			it = instance.runningSkills.erase(it);
        		}
        		else
        			++it;
        	}
        	GetQueryObject()->Execute("AnarchyOnTimer %d", g_appInfo.dwSys); // - 60k
        }
        
        bool AnarchyDBController::addSkill(const unsigned long skillId)
        {
        	Anarchy& instance = AnarchyManage::getAnarchy();
        	for (std::vector<AnarchySkillOn>::iterator it = instance.runningSkills.begin(); it !=         instance.runningSkills.end(); ++it)
        	{
        		if (it->skillId == skillId)
        		{
        			const AnarchySkillParameter& skillParam = instance.findSkillInfo(skillId);
        			if (!skillParam.isEmpty() && skillParam.isStackable())
        			{
        				it->tickEnd += skillParam.timeLasting;
        				GetQueryObject()->Execute("AnarchyUpdateTick %d, %d, %d", g_appInfo.dwSys, it->skillId,         it->tickEnd);
        				return true;
        			}
        		}
        	}
        
        	const AnarchySkillParameter& skill = instance.findSkillInfo(skillId);
        	if (!skill.isEmpty())
        	{
        		instance.runningSkills.push_back({ skillId, skill.timeLasting });
        		AnarchySkillOn& lastAddedSkill = instance.lastAddedSkill();
        		GetQueryObject()->Execute("AnarchyAddSkill %d, %d, %d", g_appInfo.dwSys, lastAddedSkill.skillId,         lastAddedSkill.tickEnd);
        		return true;
        	}
        	return false;
        }
        void AnarchyDBController::Handler(const LPDB_OVERLAPPED_PLUS pov, const DWORD dwCompletionKey)
        {
        	switch (pov->nQueryMode)
        	{
        		case eInit:
        		{
        			CDPTrans::GetInstance()->sendAnarchy(dwCompletionKey);
        			break;
        		}
        		case eAnarchySkillUse:
        		{
        			unsigned long skillId;
        			CAr ar(pov->lpBuf, pov->uBufSize);
        			ar >> skillId;
        
        			addSkill(skillId);
        			break;
        		}
        		default: break;
        	}
        }
        
        void AnarchyDBController::restoreRunningSkills()
        {
        	CQuery* pQuery	= CreateQuery();
        	if( !pQuery )
        		return;
        
        	if (!pQuery->Execute("uspAnarchyRestore %d", g_appInfo.dwSys))
        	{
        		Error("couldn't execute uspAnarchyRestore");
        		return;
        	}
        
        	Anarchy& instance = AnarchyManage::getAnarchy();
        	while (pQuery->Fetch())
        	{
        		AnarchySkillOn s;
        		s.skillId = pQuery->GetInt("id");
        		s.tickEnd = static_cast<unsigned long>(pQuery->GetInt("timeLasting"));
        		instance.runningSkills.emplace_back(std::move(s));
        		
        	}
        	SAFE_DELETE( pQuery );	
        	return;	
        }
        
        
        bool AnarchyDBController::postMessage(const int nQuery, BYTE* lpBuf, const int nBufSize, const DWORD         dwCompletionKey)
        {
        	LPDB_OVERLAPPED_PLUS pov = new DB_OVERLAPPED_PLUS;
        	if (lpBuf)
        	{
        		if (nBufSize < 0 || nBufSize >(1024 * 1024))
        		{
        			Error("PostRequest nBufSize error. nQuery[%d], nBufSize[%d]", nQuery, nBufSize);
        			if (pov){ delete pov; }
        			return false;
        		}
        		pov->lpBuf = new BYTE[nBufSize];
        		::memset(pov->lpBuf, 0, nBufSize);
        
        		::memcpy(pov->lpBuf, lpBuf, nBufSize);
        		pov->uBufSize = nBufSize;
        	}
        
        	pov->nQueryMode = nQuery;
        	const bool result = ::PostQueuedCompletionStatus(getIocp(), 1, dwCompletionKey, &pov->Overlapped);
        	return result;
        }
        
        #endif
        
        
        
        #ifdef __CLIENT
        #include "ResData.h"
        #include "DPClient.h"
        extern CDPClient g_DPlay;
        
        AnarchyWnd::AnarchyWnd() : page(0), maxPage(0)
        {
        }
        
        AnarchyWnd::~AnarchyWnd()
        {
        }
        
        BOOL AnarchyWnd::Initialize(CWndBase * pWndParent, DWORD)
        {
        	return CWndNeuz::InitDialog(g_Neuz.GetSafeHwnd(), APP_ANARCHY, 0, CPoint(0, 0), pWndParent);
        }
        
        BOOL AnarchyWnd::OnChildNotify(const UINT message, const UINT nID, LRESULT * pLResult)
        {
        	int sendAnarchy = 0;
        	switch (nID)
        	{
        		case WIDC_BUTTON1:
        			if (--page < 0)
        				page = maxPage - 1;
        			break;
        		case WIDC_BUTTON2:
        			if (++page > maxPage - 1)
        				page = 0;
        			break;
        		case WIDC_BUTTON12:
        			sendAnarchy = 1;
        			break;
        		case WIDC_BUTTON13:
        			sendAnarchy = 2;
        			break;
        		case WIDC_BUTTON14:
        			sendAnarchy = 3;
        			break;
        		case WIDC_BUTTON15:
        			sendAnarchy = 4;
        			break;
        		default: 
        			break;
        	}
        	if (sendAnarchy != 0)
        	{
        		sendAnarchy = (sendAnarchy + (page * 4)) -1;
        		const Anarchy& inst = AnarchyManage::getAnarchy();
        		if (sendAnarchy < inst.anarchyList.size())
        			g_DPlay.sendAnarchySkill(inst.anarchyList[sendAnarchy].skillId);
        		else
        			g_WndMng.PutString("Invalid.");
        
        		Destroy();
        	}
        	return CWndNeuz::OnChildNotify(message, nID, pLResult);
        }
        
        void AnarchyWnd::OnDraw(C2DRender *p2DRender)
        {
        	const Anarchy& inst = AnarchyManage::getAnarchy();
        	CRect clientRc = GetClientRect();
        	
        	size_t vecPos = page * 4; 
        	for (int nCount = 0; nCount < 4; ++nCount)
        	{
        		if (vecPos < inst.anarchyList.size())
        		{
        			if (useButtons[nCount])
        			{
        				useButtons[nCount]->EnableWindow(true);
        				useButtons[nCount]->SetVisible(true);
        			}
        
        			const AnarchySkillParameter& skillParam = inst.anarchyList[vecPos];
        			const CRect rc = skillImages[nCount]->GetWndRect();
        			const long long perinCount = skillParam.cost / 100000000;
        			const long long remaining = skillParam.cost % 100000000;
        
        			std::string costValue = std::to_string(perinCount);
        			costValue += " Perin ";
        			costValue += std::to_string(remaining);
        			costValue += " Penya";
        
        			if (g_pPlayer && g_pPlayer->GetTotalGold() > skillParam.cost)
        			{
        				p2DRender->TextOut(rc.right + 2, rc.top + 2, skillParam.name.c_str(), 0xff000000);
        				p2DRender->TextOut(rc.right + 2, rc.top + 14, costValue.c_str(), 0xff000000);
        			}
        			else
        			{
        				// Todo -> Strikeout
        				p2DRender->TextOut(rc.right + 2, rc.top + 2, skillParam.name.c_str(), 0xffff0000);
        				p2DRender->TextOut(rc.right + 2, rc.top + 14, costValue.c_str(), 0xffff0000);
        			}
        			CTexture* image = m_textureMng.AddTexture(g_Neuz.m_pd3dDevice, MakePath(DIR_ICON,         skillParam.icon.c_str()), 0xffff00ff, true);
        			if (image)
        				image->Render(p2DRender, CPoint(rc.left, rc.top), 255);
        
        			CRect rectHittest;
        			SetRect(&rectHittest, rc.left, rc.top, clientRc.right - 50, rc.bottom);
        			if (rectHittest.PtInRect(m_ptMouse))
        			{
        				CPoint pt2 = m_ptMouse;
        				ClientToScreen(&pt2);
        				ClientToScreen(&rectHittest);
        
        				CEditString editString;
        				editString.AddString(skillParam.name.c_str(), D3DCOLOR_XRGB(0, 93, 0), ESSTY_BOLD);
        
        				const long long buffTimer = skillParam.timeLasting / 1000;
        				CTimeSpan timeSpan(0, 0, 0, buffTimer);
        				std::string tmp = "\nTime Remaining: ";
        				tmp += std::to_string(timeSpan.GetHours());
        				tmp += "h, ";
        				tmp += std::to_string(timeSpan.GetMinutes());
        				tmp += "m, ";
        				tmp += std::to_string(timeSpan.GetSeconds());
        				tmp += "s";
        				editString.AddString(tmp.c_str(), 0xff000000);
        
        				tmp = "\n";
        				tmp += skillParam.description;
        
        				editString.AddString(tmp.c_str(), 0xFF323232);
        				g_toolTip.PutToolTip(123456789, editString, rectHittest, pt2, 1);
        			}
        			++vecPos;
        		}
        		else
        		{
        			if (useButtons[nCount])
        			{
        				useButtons[nCount]->EnableWindow(false);
        				useButtons[nCount]->SetVisible(false);
        			}
        			
        		}
        	}
        
        	CRect rect = GetClientRect();
        	std::string temp = std::to_string(page + 1);
        	temp += " / ";
        	temp += std::to_string(maxPage);
        	p2DRender->TextOut((rect.right / 2) - 5, rect.bottom - 20, temp.c_str(), 0xff000000);
        }
        
        void AnarchyWnd::OnInitialUpdate()
        {
        	CWndNeuz::OnInitialUpdate();
        
        	const Anarchy& inst = AnarchyManage::getAnarchy();
        	maxPage = (inst.listSize() / 4) + ((inst.listSize()) % 4 > 0 ? 1 : 0);
        
        	for (int i = 0; i < 4; ++i)
        		useButtons[i] = dynamic_cast<CWndButton*>(GetDlgItem(WIDC_BUTTON12 + i));
        
        	pageButtons[0] = dynamic_cast<CWndButton*>(GetDlgItem(WIDC_BUTTON1));
        	pageButtons[1] = dynamic_cast<CWndButton*>(GetDlgItem(WIDC_BUTTON2));
        
        	skillImages[0] = dynamic_cast<CWndStatic*>(GetDlgItem(WIDC_STATIC));
        	skillImages[1] = dynamic_cast<CWndStatic*>(GetDlgItem(WIDC_STATIC1));
        	skillImages[2] = dynamic_cast<CWndStatic*>(GetDlgItem(WIDC_STATIC2));
        	skillImages[3] = dynamic_cast<CWndStatic*>(GetDlgItem(WIDC_STATIC3));
        	skillImages[0]->SetVisible(false);
        	skillImages[1]->SetVisible(false);
        	skillImages[2]->SetVisible(false);
        	skillImages[3]->SetVisible(false);
        	MoveParentCenter();
        }
        #endif
        #endif
        ```
       
        
## Database changes
---
- Create a new Table in CHARACTER_01_DBF
    ```SQL
    use [CHARACTER_01_DBF]
    go

    CREATE TABLE anarchySkills(
        sServer int NOT NULL,
        id int not null,
        timeLasting int not null,
    );
    ```
- **AnarchyAddSkill**
    ```SQL
    USE [CHARACTER_01_DBF]
    SET ANSI_NULLS ON
    SET QUOTED_IDENTIFIER ON
    GO
    CREATE PROCEDURE [dbo].[AnarchyAddSkill]
    	@server int,
    	@skillId int,
    	@timeLasting int
    AS
    BEGIN
    	SET NOCOUNT ON;
    
         INSERT INTO dbo.anarchySkills(sServer, id, timeLasting)
    	 VALUES ( @server, @skillId, @timeLasting )
    
    END
    ```
- **AnarchyOnTimer**
    ```SQL
    USE [CHARACTER_01_DBF]
    SET ANSI_NULLS ON
    SET QUOTED_IDENTIFIER ON
    GO
    CREATE PROCEDURE [dbo].[AnarchyOnTimer]
    	@server int
    AS
    BEGIN
    	SET NOCOUNT ON;
    
    	delete from dbo.anarchySkills
    	where timeLasting - 60000 < 0
    
    	UPDATE dbo.anarchySkills
    	SET timeLasting = timeLasting - 60000
    	where sServer = @server
    END
    ```
- **AnarchyRemoveSkill**
    ```SQL
    USE [CHARACTER_01_DBF]
    SET ANSI_NULLS ON
    SET QUOTED_IDENTIFIER ON
    GO
    CREATE PROCEDURE [dbo].[AnarchyRemoveSkill]
    	@server int,
    	@skillId int
    AS
    BEGIN
    	SET NOCOUNT ON;
    
    	delete from dbo.anarchySkills
    	where sServer = @server and id = @skillId
    END
    ```
- **AnarchyUpdateTick**
    ```SQL
    USE [CHARACTER_01_DBF]
    SET ANSI_NULLS ON
    SET QUOTED_IDENTIFIER ON
    GO
    CREATE PROCEDURE [dbo].[AnarchyUpdateTick]
    	@server int,
    	@skillId int,
    	@timeLasting int
    AS
    BEGIN
    	SET NOCOUNT ON;
    	
    	update dbo.anarchySkills 
    	set timeLasting = @timeLasting 
    	where sServer = @server and id = @skillId
    END
    ```
- **uspAnarchyRestore**
    ```SQL
    USE [CHARACTER_01_DBF]
    SET ANSI_NULLS ON
    SET QUOTED_IDENTIFIER ON
    GO
    
    CREATE PROCEDURE [dbo].[uspAnarchyRestore] 
    @nServer int
    as
    begin
    	set nocount on
    	select id, timeLasting 
    	from anarchySkills
    	where sServer= @nServer
    end
    ```

## Resource Edits + Default File
---
- Create a new file-> **Anarchy.txt**
    ```CPP
    AnarchySkills
    {
    	1 "Haste of the Powerful" 10 1800000 11 15 0 0 "Speed +15%" "Icon_Anarchy1.png" 0
    	2 "Enchanted Strength" 1000000000 1800000 10003 10 0 0 "All stats +10" "Icon_Anarchy2.png" 0
    	3 "Bountiful Treasure" 1 1800000 1003 10 0 0 "Item Drop Rate +10%" "Icon_Anarchy3.png" 0
    	4 "Boost of the Mentor" 1 1800000 1006 15 0 0 "Experience +15%" "Icon_Anarchy4.png" 0
    	5 "Hand of Midas" 1 1800000 1004 20 0 0 "Penya Drop +20%" "Icon_Anarchy5.png" 0
    	6 "Time Warp" 1 1800000 1001 20 0 0 "All dungeon cooldowns are reduced by 20% while this is active"        "Icon_Anarchy6.png" 0
    	7 "Divine Loot" 1 1800000 1002 10 0 0 "Increased Drop Rate of Materials +10%" "Icon_Anarchy7.png" 0
    	8 "Gifted Extractor" 1 1800000 1005 20 0 0 "Decreased Collecting Time +20%	" "Icon_Anarchy8.png" 0
    	9 "Powerful Bond" 1 1800000 1000 15 0 0 "Increased Pet Experience +15%" "Icon_Anarchy9.png" 0
    }
    ```
- **DefineNeuz.h**
    ```
    #define MMI_Anarchy 314
    ```
- **DefineText.h**
    ```
    #define TID_MMI_Anarchy 7314
    ```
- **ResData.h**
    ```
    #define APP_ANARCHY 2201
    ```
- **Resdata.inc**
    ```
    APP_ANARCHY "WndTile00.tga" "" 1 352 256 0x2410000 26
    {
    // Title String
    IDS_RESDATA_INC_090030
    }
    {
    // ToolTip
    IDS_RESDATA_INC_009611
    }
    {
        WTYPE_STATIC WIDC_STATIC "" 0 30 20 61 51 0x2220002 0 0 0 0 13258528 13237536 13255752
        {
        // Title String
        IDS_RESDATA_INC_009611
        }
        {
        // ToolTip
        IDS_RESDATA_INC_009611
        }
        WTYPE_STATIC WIDC_STATIC1 "" 0 30 62 61 93 0x2220002 0 0 0 0 6750324 13101364 3211296
        {
        // Title String
        IDS_RESDATA_INC_009611
        }
        {
        // ToolTip
        IDS_RESDATA_INC_009611
        }
        WTYPE_STATIC WIDC_STATIC2 "" 0 30 100 61 131 0x2220002 0 0 0 0 6750324 13101364 3211296
        {
        // Title String
        IDS_RESDATA_INC_009611
        }
        {
        // ToolTip
        IDS_RESDATA_INC_009611
        }
        WTYPE_STATIC WIDC_STATIC3 "" 0 30 140 61 171 0x2220002 0 0 0 0 6750324 13101364 3211296
        {
        // Title String
        IDS_RESDATA_INC_009611
        }
        {
        // ToolTip
        IDS_RESDATA_INC_009611
        }
        WTYPE_BUTTON WIDC_BUTTON1 "Buttleft2.bmp" 0 23 193 64 214 0x220010 0 0 0 0 0 0 0
        {
        // Title String
        IDS_RESDATA_INC_009611
        }
        {
        // ToolTip
        IDS_RESDATA_INC_009611
        }
        WTYPE_BUTTON WIDC_BUTTON2 "Buttright2.bmp" 0 270 193 314 214 0x220010 0 0 0 0 0 0 0
        {
        // Title String
        IDS_RESDATA_INC_009611
        }
        {
        // ToolTip
        IDS_RESDATA_INC_009611
        }
        WTYPE_BUTTON WIDC_BUTTON12 "ButtPay.BMP" 0 274 20 319 41 0x220010 0 0 0 0 0 0 0
        {
        // Title String
        IDS_RESDATA_INC_009611
        }
        {
        // ToolTip
        IDS_RESDATA_INC_009611
        }
        WTYPE_BUTTON WIDC_BUTTON13 "ButtPay.BMP" 0 274 64 318 85 0x220010 0 0 0 0 0 0 0
        {
        // Title String
        IDS_RESDATA_INC_009611
        }
        {
        // ToolTip
        IDS_RESDATA_INC_009611
        }
        WTYPE_BUTTON WIDC_BUTTON14 "ButtPay.BMP" 0 274 110 318 129 0x220010 0 0 0 0 0 0 0
        {
        // Title String
        IDS_RESDATA_INC_009611
        }
        {
        // ToolTip
        IDS_RESDATA_INC_009611
        }
        WTYPE_BUTTON WIDC_BUTTON15 "ButtPay.BMP" 0 274 154 318 175 0x220010 0 0 0 0 0 0 0
        {
        // Title String
        IDS_RESDATA_INC_009611
        }
        {
        // ToolTip
        IDS_RESDATA_INC_009611
        }
    }
    ```
- **Resdata.txt.txt**
    ```
    IDS_RESDATA_INC_090030 Anarchy System - Buff Lists
    ```
- **Textclient.inc**
    ```
    TID_MMI_Anarchy 0xffffffff
    {
	    IDS_TEXTCLIENT_INC_040001
    }
    ```
- **Textclient.txt**
    ```
    IDS_TEXTCLIENT_INC_040001 Anarchy
    ```
- **[Icons](https://drive.google.com/file/d/1IaQXdfSlqKOTe2jUoE9PXz2GmFW9BaGA/view?usp=sharing)** (_Ripped from cwflyff_)
    
## Png Format
---
- If you are having issues with icons not loading, that means you do not open png files. Just change the files to tga or dds, or I can charge for a png loader.

## Changes based on VS
---
_if you run an older compiler_
- Remove [[nodiscard]] (_This is used in later compiler versions so if it errors for you_)
- [updated] now static inline construction is changed depending on CXX ver

## Thank you
---
_FlyFF is owned and operated and developed by Galanet/Aeonsoft. This is only an implementation made by myself, Kia#1411._

[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen. Thanks SO - http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)

