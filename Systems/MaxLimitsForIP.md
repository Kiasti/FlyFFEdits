## Limits for IP
---
This was made because of the idea to limit collectors to an amount per IP, but more than that can be done or even added.

#### AccountServer implementation
---
This part will limit additional logins.


- MsgHdr.h
```cpp
#ifdef __MaxIpLimits
const unsigned char account_tooManyLogged = 10;
#endif
```

- DPCertified.cpp (Neuz)
```cpp
		case ERROR_FLYFF_EXPIRED_SESSION_PASSWORD:
			nText	= TID_ERROR_EXPIRED_SESSION_PASSWORD;
			break;
#ifdef __MaxIpLimits
		case Error_MaxIpReached:
			nText = TID_GAME_LIMITCONNECTION;
			break;
#endif
```

- DPSrvr.h (Accountserver)
```cpp
#ifdef __MaxIpLimits
	CMclCritSec m_csIpLists;
	std::unordered_map<unsigned long, int> ipToCount;
#endif
```

- DpSrvr.cpp (Accountserver)
<small>*CDPSrvr::OnAddAccount*</small>

```cpp
	#ifdef __MaxIpLimits
		const char* str = lpAddr;
		unsigned long start = 0;
		while (*str) {
			start = start * 10 + (*str++ - '0');
		}
	#endif
```

```cpp
#ifdef __MaxIpLimits
			cbResult = g_AccountMng.AddAccount(lpszAccount, dpid1, dpid2, &dwAuthKey, cbAccountFlag, start);
#else
			cbResult = g_AccountMng.AddAccount( lpszAccount, dpid1, dpid2, &dwAuthKey, cbAccountFlag, 0 );
#endif
```
> Note: <small>Pretty sure fCheck is not used. At least, not in any of the sources I have used, so you can just replace the variable in the implementation</small>

- Account.cpp

```cpp

#ifdef __MaxIpLimits
BYTE CAccountMng::AddAccount(LPCTSTR lpszAccount, DWORD dpid1, DWORD dpid2, DWORD* pdwAuthKey, BYTE cbAccountFlag, unsigned long ip)
#else
BYTE CAccountMng::AddAccount(LPCTSTR lpszAccount, DWORD dpid1, DWORD dpid2, DWORD* pdwAuthKey, BYTE cbAccountFlag, int fCheck)
#endif
{
	CMclAutoLock Lock( m_AddRemoveLock );
#ifdef __MaxIpLimits
	CMclAutoLock Lock2(m_csIpLists);
	const auto iter = ipToConnectedCount.find(ip);
	const bool b = iter != ipToConnectedCount.end();
	if (b && iter->second >= maxConnections)
		return account_tooManyLogged;
#endif

	std::map<std::string, CAccount*>::iterator i1	= m_stringToPAccount.find( lpszAccount );
	if( i1 == m_stringToPAccount.end() )
	{
		int nIndex	= GetIndex( dpid1 );
		if( nIndex >= 0 && nIndex < MAX_CERTIFIER )
		{
			std::map<DWORD, CAccount*>::iterator i2	= m_adpidToPAccount[nIndex].find( dpid2 );
			if( i2 == m_adpidToPAccount[nIndex].end() )
			{
#ifdef __MaxIpLimits
				if (b) { iter->second++; }
				else { ipToConnectedCount.insert(std::make_pair(ip, 1)); }
				CAccount* pAccount = new CAccount(lpszAccount, dpid1, dpid2, cbAccountFlag, ip);
#else
				CAccount* pAccount = new CAccount(lpszAccount, dpid1, dpid2, cbAccountFlag, fCheck);

#endif
				m_stringToPAccount.insert(std::map<std::string, CAccount*>::value_type( lpszAccount, pAccount ) );
				m_adpidToPAccount[nIndex].insert(std::map<DWORD, CAccount*>::value_type( dpid2, pAccount ) );
				m_nCount++;
				*pdwAuthKey = pAccount->m_dwAuthKey = xRandom(0x00000001UL, ULONG_MAX);
				pAccount->m_bCertify = TRUE;
				return ACCOUNT_CHECK_OK;
			}
			return ACCOUNT_DUPLIACTE;
		}
		return ACCOUNT_DUPLIACTE;
	}
	else
	{
		CAccount* pAccount	= GetAccount( lpszAccount );
		if( pAccount && FALSE == pAccount->m_fRoute )
		{
			if( timeGetTime() - pAccount->m_dwPing > MIN( 2 ) )
			{
				g_dpSrvr.DestroyPlayer( pAccount->m_dpid1, pAccount->m_dpid2 );
				RemoveAccount( pAccount->m_dpid1, pAccount->m_dpid2 );
#ifdef __MaxIpLimits
				return AddAccount(lpszAccount, dpid1, dpid2, pdwAuthKey, cbAccountFlag, ip);
#else
				return AddAccount(lpszAccount, dpid1, dpid2, pdwAuthKey, cbAccountFlag, fCheck);
#endif
			}
		}
		return ACCOUNT_DUPLIACTE;
	}
}
```

```cpp
void CAccountMng::RemoveAccount( LPCTSTR lpszAccount )
{
	CMclAutoLock	Lock( m_AddRemoveLock );
#ifdef __MaxIpLimits
	CMclAutoLock Lock2(m_csIpLists);
#endif

// emited code
#ifdef __MaxIpLimits
		if (const auto iter = ipToConnectedCount.find(pAccount->GetIP()); iter != ipToConnectedCount.end() && iter->second-- < 1) 
		 ipToConnectedCount.erase(iter);
#endif
		g_DbManager.UpdateTracking( FALSE, lpszAccount );	// À¯Àú°¡ 

```


- Account.h 
<small>in CAccount</small>

```cpp
#ifdef __MaxIpLimits	
	private:
		unsigned long connectedIp;
	public:
		unsigned long GetIP() const { return connectedIp; }
#endif
```

<small> in CAccountMng</small>

```cpp
	std::map<DWORD, CAccount*>	m_adpidToPAccount[MAX_CERTIFIER];
#ifdef __MaxIpLimits
	CMclCritSec m_csIpLists;
	std::map<unsigned long, unsigned long> ipToConnectedCount;
#endif
```
```cpp
#ifdef __MaxIpLimits
	unsigned long maxConnections = NULL_ID;
#endif
```
```cpp
#ifdef __MaxIpLimits
	BYTE AddAccount(LPCTSTR lpszAccount, DWORD dpid1, DWORD dpid2, DWORD* pdwAuthKey, BYTE b18, unsigned long ip);
#else
	BYTE	AddAccount( LPCTSTR lpszAccount, DWORD dpid1, DWORD dpid2, DWORD* pdwAuthKey, BYTE b18, int fCheck );
#endif
```

<br>

#### Worldserver Implementation
---
This part contains a lot more limitations

- DPsrvr.cpp
<small>*CDPSrvr::OnAddUser*</small>

```cpp
	pUser->m_dwAuthKey = dwAuthKey;
	memcpy( pUser->m_playAccount.lpAddr, lpAddr, 16 );

#ifdef __MaxIpLimits
	g_UserMng.AddIpList(pUser);
#endif
```

<small>*CDPSrvr::OnPVendorOpen*</small>

```cpp
#ifdef __MaxIpLimits
		if (!g_UserMng.checkIpAction(pUser, CUserMng::maxIpType::shopOpen))
		{
			pUser->AddText("Too many shop accounts on. Sorry ):");
			return;
		}
		g_ChattingMng.NewChattingRoom(pUser->m_idPlayer);
#endif
```
> Note: <small>If you're using Florist base, it'd be the first thing within the bOwnShop scope. And either before or after the mapPlayerShops check.</small>


<small>*CDPSrvr::OnQueryStartCollecting*</small>

```cpp
#ifdef __MaxIpLimits
		if (g_UserMng.checkIpAction(pUser, CUserMng::maxIpType::collecting))
			pUser->StartCollecting();
		else
			pUser->AddText("Sorry, but you already have the maximum amount of collectors on!");
#else
		pUser->StartCollecting();
#endif
```


- User.cpp
```cpp
CUserMng::CUserMng()
{
	m_lCount = 0;
#ifdef __MaxIpStuff
	maxIpSettings.options[std::to_underlying(maxIpType::collecting)] = 2;
	maxIpSettings.options[std::to_underlying(maxIpType::dungeonEnter)] = 4;
	maxIpSettings.options[std::to_underlying(maxIpType::dungeoningDropRate)] = 3;
	maxIpSettings.options[std::to_underlying(maxIpType::shopOpen)] = 3;
#endif
}
```

<small>*CUserMng::RemoveUser*</small>

```cpp
		--m_lCount;
#ifdef __MaxIpLimits
		removeIpList(pUser);
#endif
```

```cpp

#ifdef __MaxIpLimits
void CUserMng::AddIpList(CUser* pUser)
{	
	if (const auto iter = playersToIps.find(pUser->getUserIp()); iter == playersToIps.end())
	{
		std::vector<CUser*> temp = { pUser };
		playersToIps.insert(std::make_pair(pUser->getUserIp(), std::move(temp)));
	}
	else
		iter->second.push_back(pUser);
}

void CUserMng::removeIpList(CUser* pUser)
{	
	if (const auto iter = playersToIps.find(pUser->getUserIp()); iter != playersToIps.end())
	{
		const auto iter2 = std::find_if(iter->second.begin(), iter->second.end(), [pUser](const CUser const* user1) -> const bool {if (user1 == pUser) return true; return false; });
		if (iter2 != iter->second.end())
			playersToIps.erase(iter);
	}
}

bool CUserMng::checkIpAction(CUser* pUser, maxIpType ipLimit)
{
	if (ipLimit >= maxIpType::max)
		return true;

	auto iter = playersToIps.find(pUser->getUserIp());
	if (iter != playersToIps.end())
	{
		unsigned char amount = 0;
		for (auto iterz = iter->second.begin(); iterz != iter->second.end(); ++iterz)
		{
			if (pUser == *iterz && ipLimit != maxIpType::dungeoningDropRate)
				continue;

			switch (ipLimit)
			{
				case maxIpType::collecting:
					if ((*iterz)->IsCollecting())
						amount++;
					break;
				case maxIpType::dungeonEnter:
				case maxIpType::dungeoningDropRate:
				{
					//not same dungeon
					CWorld* pWorld = pUser->GetWorld();
					CWorld* iterzWorld = (*iterz)->GetWorld();
					if (pWorld && iterzWorld && pWorld != iterzWorld)
					{
						if (CInstanceDungeonHelper::GetInstance()->IsInstanceDungeon(iterzWorld->GetID()))
							amount++;
					}
					break;
				}
				case maxIpType::shopOpen:
					if ((*iterz)->m_vtInfo.IsVendorOpen())
						amount++;
					break;
				default:
					break;
			}
		}
		if (amount >= maxIpSettings.options[std::to_underlying(ipLimit)])
			return false;
	}
	return true;
}
#endif
```

- User.h

<small>*CUser*</small>

```cpp
#ifdef __MaxIpLimits
	char* getUserIp() { return m_playAccount.lpAddr; }
#endif
```

<small>*CUserMng*</small>

```cpp
#ifdef __MaxIpLimits
	enum class maxIpType : unsigned char { collecting, dungeonEnter, dungeoningDropRate, shopOpen, max };
	struct MaxIPPerSetting {
		unsigned char options[std::to_underlying(maxIpType::max)] = { 0 };
		MaxIPPerSetting() {}
	};
	MaxIPPerSetting maxIpSettings;

	private:
		struct cmpCharPtr { bool operator()(char const* a, char const* b) const { return std::strcmp(a, b) < 0; } };
		std::unordered_map<std::string, std::vector<CUser*>> playersToIps;

	public:
		void AddIpList(CUser* pUser);
		void removeIpList(CUser* pUser);
		bool checkIpAction(CUser* pUser, maxIpType ipLimit = maxIpType::max);
#endif
```

- InstanceDungeonParty.h
```cpp
#ifdef __MaxIpLimits
extern CUserMng g_UserMng;
#endif
BOOL CInstanceDungeonParty::EnteranceDungeon( CUser* pUser, DWORD dwWorldId )
{
#ifdef __MaxIpLimits
	if (g_UserMng.checkIpAction(pUser, CUserMng::maxIpType::dungeonEnter))
	{
		pUser->AddText("You've reached the maximum amount of characters allowed per ip address.");
		return false;
	}
#endif

	if( pUser->m_idparty == 0)
	{
		pUser->AddDefinedText(TID_GAME_INSTANCE_PARTY);
		return false;
	}
	return TeleportToDungeon( pUser, dwWorldId, pUser->m_idparty );
}
```
> Note: <small>*Because florists base added solo dungeons, I thought it was more wise to move the implementation to CInstanceDungeonParty instead of the CInstanceDungeonHelper class*</small>


- Mover.cpp
CMover::OnDrop

```cpp
				else { nProbability = 10;	nPenyaRate = 50; }
			}

#ifdef __MaxIpLimits
			if (changeProb)
				nProbability = 10;
#endif

// emited code

#ifdef __MaxIpLimits
					if ((lpDropItem = lpMoverProp->m_DropItemGenerator.GetAt(i, bUnique, GetPieceItemDropRateFactor(pAttacker) * maxIpDropFactor)) != NULL)
#else
					if ((lpDropItem = lpMoverProp->m_DropItemGenerator.GetAt(i, bUnique, GetPieceItemDropRateFactor(pAttacker))) != NULL)
#endif

//emited code

						DWORD dwPrabability = (DWORD)(prj.m_adwExpDropLuck[(pItemProp->dwItemLV > 120 ? 119 : pItemProp->dwItemLV - 1)][k]
							* ((float)lpMoverProp->dwCorrectionValue / 100.0f));

#ifdef __MaxIpLimits
						dwPrabability *= maxIpDropFactor;
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

