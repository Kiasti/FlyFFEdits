## No Party Required to Make Guild
----
Another thing I did in Simple. 

- NpcScript.cpp <small>(*WorldDialog*)</small>
```cpp

#ifdef __NoPartyToMakeGuild
void CNpcScript::masa_helena_1()
{
	if (IsGuild() == 0)
	{
		if (GetPlayerGold() >= 3000000)
		{
			RemoveGold(3000000);
			CreateGuild();
			Say(752);
			EndQuest(QUEST_CREGUILD);
		}
		else
			Say(755);
	}
	else
		Say(754);
}

#else
// oldcode
#endif
```
> Note: There is more changes within this file, but I am pretty sure this is the only one required. If issues happen, contact me.


- Scriptlib.cpp
```cpp

int APIENTRY CreateGuild(NPCDIALOG_INFO* pInfo)
{
#ifdef __NoPartyToMakeGuild
	const CUser* pUser = prj.GetUser(pInfo->GetPcId());
	GUILD_MEMBER_INFO info;
	info.idPlayer = pUser->m_idPlayer;
	g_DPCoreClient.SendCreateGuild(&info, 1, "");
	return 1;
#else
	const CUser* pUser = prj.GetUser(pInfo->GetPcId());
	int nSize = 0;
	GUILD_MEMBER_INFO info[MAX_PTMEMBER_SIZE];

	if (CParty* pParty = g_PartyMng.GetParty(pUser->m_idparty); pParty && pParty->IsLeader(pUser->m_idPlayer))
	{
		for (int i = 0; i < pParty->m_nSizeofMember; i++)
		{
			CUser* pUsertmp = g_UserMng.GetUserByPlayerID(pParty->GetPlayerId(i));
			if (IsValidObj(pUsertmp))
			{
				info[nSize].idPlayer = pUsertmp->m_idPlayer;
				nSize++;
			}
			else
				return 0;
		}
	}
	else
		return 0;

	g_DPCoreClient.SendCreateGuild(info, nSize, "");
	return 1;
#endif
}
```
