
## AutoTarget
---
The system will auto target the next mob that is attacking you / aggro'd to you. There might be issues with some skills.



- Msghdr.h
```cpp
#ifdef __AutoTarget
#define SNAPSHOTTYPE_AAON static_cast<unsigned short>(0xff57)
#endif
```

- Dpclient.cpp
```cpp
#ifdef __AutoTarget
			case SNAPSHOTTYPE_AAON:					setAAOn(ar); break;
#endif

```

```cpp
#ifdef __AutoTarget
void CDPClient::setAAOn(CAr& ar)
{
	unsigned long id = NULL_ID;
	ar >> id;
	if (g_pPlayer && g_Option.useAutoTarget)
	{
		CWorld* pWorld = g_pPlayer->GetWorld();
		CMover* enemey = prj.GetMover(id);
		if (pWorld && IsValidObj(enemey))
		{
			pWorld->SetObjFocus(enemey);
			g_WndMng.m_pWndWorld->m_bAutoAttack = true;
		}
	}
}
#endif
```

- dpClient.h
```cpp
#ifdef __AutoTarget
	void setAAOn(CAr& ar);
#endif
```

- AttackArbiter.cpp
```cpp


#ifdef __AutoTarget
	bool isViable = true;
	if (m_dwAtkFlags & AF_MELEESKILL || m_dwAtkFlags & AF_MAGICSKILL)
	{
		const int skillId = (m_nParam >> 8) & 0xFF;
		const int skillLevel = (m_nParam >> 16) & 0xFFFF;
		ItemProp* pSkillProp;
		AddSkillProp *pAddSkillProp;
		bool real = m_pAttacker->GetSkillProp(&pSkillProp, &pAddSkillProp, skillId, skillLevel, "");
		if (real && pAddSkillProp->dwSkillRange != NULL_ID) { isViable = false; }
	}
	else if (m_dwAtkFlags & AF_FORCE) { isViable = false; }

	if (isViable)
	{
		CWorld* pWorld = m_pAttacker->GetWorld();
		if (pWorld && m_pAttacker->IsPlayer())
		{
			const float fRange = 5.0f;
			CObj* pObj;
			bool found = false;
			FOR_LINKMAP(pWorld, m_pAttacker->GetPos(), pObj, 1, CObj::linkDynamic, m_pAttacker->GetLayer())
			{
				if (pObj && pObj->GetType() == OT_MOVER)
				{
					CMover* pMover = static_cast<CMover*>(pObj);
					if (pMover != m_pAttacker && pMover != m_pDefender)
					{
						MoverProp* mProp = pMover->GetProp();
						if (!mProp || mProp->dwAI == AII_PET || mProp->dwAI == AII_EGG)
							continue;
						if (pObj->IsRangeObj(m_pAttacker->GetPos(), fRange))
						{
							if (pMover->GetDestId() == m_pAttacker->GetId())
							{
								m_pAttacker->SetDestObj(pMover->GetId(), fRange, true);
								dynamic_cast<CUser*>(m_pAttacker)->AutoAttackOn(pMover->GetId());
								found = true;
							}
						}
					}
				}
			}
			END_LINKMAPBREAK(found)
		}
	}
#endif
```

- User.cpp
```cpp
#ifdef __AutoTarget
void CUser::AutoAttackOn(unsigned long obji)
{
	m_Snapshot.cb++;
	m_Snapshot.ar << GetId();
	m_Snapshot.ar << SNAPSHOTTYPE_AAON;
	m_Snapshot.ar << obji;
}
#endif
```

- User.h
```cpp
#ifdef __AutoTarget
	void AutoAttackOn(unsigned long objid);
#endif
```

- hwOption.cpp
```cpp
#ifdef __AutoTarget
	useAutoTarget = true;
#endif
```
```cpp
#ifdef __AutoTarget
		else if (scan.Token == _T("AutoTarget"))
			useAutoTarget = scan.GetNumber() != 0;
#endif
```
```cpp
#ifdef __AutoTarget
	_ftprintf(fp, _T("AutoTarget %d\n"), static_cast<int>(useAutoTarget));
#endif
```

- HwOption.h
```cpp
#ifdef __AutoTarget
		bool useAutoTarget;
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


