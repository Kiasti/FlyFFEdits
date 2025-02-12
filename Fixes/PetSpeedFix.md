## PetSpeedFix
---
This should make it so CS pets match the user's speed. 


- CMover::GetSpeed
```cpp
#ifdef __CLIENT
	if (m_dwAIInterface == AII_EGG && m_pAIInterface)
	{
		CAIEgg* pAI = (CAIEgg*)m_pAIInterface;
		CMover* pOwner = prj.GetMover(pAI->GetOwnerId());
		if (IsValidObj(pOwner))
				return pOwner->GetSpeed(pOwner->m_pActMover->m_fSpeed);
		}
#endif	// __CLIENT

#ifdef __PetSpeedFix
	if (const auto petId = GetEatPetId(); !m_bPlayer && petId != NULL_ID)
	{
		if (CMover* pOwner = prj.GetMover(petId); IsValidObj(pOwner) && pOwner->m_pActMover)
		{
			if (pOwner->IsFly())
				return pOwner->GetSpeed(pOwner->m_pActMover->m_fSpeed) * 3; // todo: grab flight speed
			return pOwner->GetSpeed(pOwner->m_pActMover->m_fSpeed);
		}
	}
```

- CMover::ActivateEatPet

```cpp
		if (pAIPet)
		{
			pAIPet->SetOwner(GetId());
			SetEatPetId(pEatPet->GetId());
#ifdef __PetSpeedFix // set the eat pets eatpet id to owner lmao. you are now the eat pet nerd.
			pEatPet->SetEatPetId(m_objid);
#endif

```

- objSerailizeOpt.cpp
```cpp
#if __VER >= 9	//__AI_0509
			ar >> m_fSpeedFactor;
#endif	// __AI_0509

#ifdef __PetSpeedFix
			unsigned long ownerId;
#if !defined(__DBSERVER)
			ar >> ownerId;
			SetEatPetId(ownerId);
#else
			ar >> ownerId;
#endif
#endif
```
```cpp
#if __VER >= 9	//__AI_0509
			ar << m_fSpeedFactor;
#endif	// __AI_0509
#ifdef __PetSpeedFix 
#if !defined(__DBSERVER)
			ar << GetEatPetId();
#else
			ar << 0ul;
#endif
#endif
```

