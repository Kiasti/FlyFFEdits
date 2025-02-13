## Dark Illusion fixes
---


[![Preview](http://img.youtube.com/vi/stHHCEiZkMI/0.jpg)](https://www.youtube.com/watch?v=stHHCEiZkMI)



CMover::Render
```cpp
#ifdef __DARKILLUSION_FIX
	bool ObjectTransparent = HasBuff(BUFF_SKILL, SI_ACR_SUP_DARKILLUSION) || IsMode(TRANSPARENT_MODE);
	bool CanSeeThroughInvis = IsActiveMover() || g_pPlayer->IsAuthHigher(AUTH_GAMEMASTER);
	if (ObjectTransparent && CanSeeThroughInvis)
	{
		if (m_pBalloonFlag && m_pBalloon)
			m_pBalloon->SetBlendFactor(80);
		if (m_pAngel&& m_pAngelFlag)
		{
			m_pAngel->m_nNoEffect = m_pModel->m_nNoEffect;
			m_pAngel->Render(pd3dDevice, &m_AngelWorldM);
			m_pAngel->m_nNoEffect = 0;
		}
		if (m_pBalloonFlag && m_pBalloon != NULL)
		{
			m_pBalloon->m_nNoEffect = m_pModel->m_nNoEffect;
			m_pBalloon->Render(pd3dDevice, &m_BalloonWorldM);
			m_pBalloon->m_nNoEffect = 0;
		}

		m_pModel->SetBlendFactor(80);
		m_pModel->Render(pd3dDevice, &mWorld);
	}
	else if (!ObjectTransparent)
	{
#ifdef __YSMOOTH_OBJ
		if (CWorld* MonstersWorld = GetWorld(); m_bSmooth && MonstersWorld && !MonstersWorld->IsPvPWorld())
		{
			if (m_pModel->m_nNoEffect == 2)
				pModel->SetBlendFactor(255);
			else
			{
				m_nSmoothCount += 2;
				if (m_nSmoothCount > 255)
				{
					m_nSmoothCount = 0;
					m_bSmooth = false;
					pModel->SetBlendFactor(255);
				}
				pModel->SetBlendFactor(m_nSmoothCount);
				
			}
		}
		else
#endif 
		{
			if (m_pBalloon)
				m_pBalloon->SetBlendFactor(255);
			if (m_pAngel)
				m_pAngel->SetBlendFactor(255);

			m_pModel->SetBlendFactor(255);
		}
		m_pModel->Render(pd3dDevice, &mWorld);

	}

	if (!IsActiveMover())
	{
		CMover* tempMover = prj.GetMover(g_pPlayer->GetEatPetId());
		if (IsValidObj(tempMover) && tempMover == this)
			m_pModel->Render(pd3dDevice, &mWorld);

		CMover* pPet = g_pPlayer->m_pet.GetObj();
		if (IsValidObj(pPet) && pPet == this)
			m_pModel->Render(pd3dDevice, &mWorld);
	}
#else
	if( IsMode( TRANSPARENT_MODE ) )		// Åõ¸í»óÅÂ°¡ ¾Æ´Ò¶§¸¸ ·»´õ.
	{
		if( !IsAuthHigher( AUTH_GAMEMASTER ) )	// ÀÏ¹Ý »ç¿ëÀÚ,
		{
			m_pModel->Render( pd3dDevice, &mWorld ); 
		}
		else
		{
			m_pModel->SetBlendFactor( 80 );
			if( IsActiveMover() ||		// ÀÚ±âÀÚ½ÅÀº ¹ÝÅõ¸íÀ¸·Î Ãâ·Â ...È¤Àº
				(IsActiveMover() == FALSE && g_pPlayer->IsAuthHigher( AUTH_GAMEMASTER )) )		// Å¸ÀÎÀÎµ¥ ÇÃ·¹ÀÌ¾î°¡ °×¸¶¸é.
				m_pModel->Render( pd3dDevice, &mWorld );		// ¹ÝÅõ¸íÀ¸·Î Ãâ·Â
			m_pModel->SetBlendFactor( 255 );
		}
	}
	else
	{
#ifdef __YSMOOTH_OBJ
		if (m_bSmooth && m_pModel->m_nNoEffect != 2)
		{
			m_nSmoothCount += 2;

			if (m_nSmoothCount > 255)
			{
				m_wBlendFactor = 255;
				m_nSmoothCount = 0;
				m_bSmooth = false;
			}
			else
				m_wBlendFactor = m_nSmoothCount;
		}
#endif //__YSMOOTH_OBJ

		m_pModel->SetBlendFactor(m_wBlendFactor);
		m_pModel->Render(pd3dDevice, &mWorld);	// ÀÏ¹Ý »óÅÂ Ãâ·Â
	}

#endif
```

CMover::Process
```cpp

#ifdef __DARKILLUSION_FIX
#ifdef __CLIENT

		CMover* pPet = m_pet.GetObj();
		if (pPet)
		{
			if (HasBuff(BUFF_SKILL, SI_ACR_SUP_DARKILLUSION))
				pPet->m_dwMode |= TRANSPARENT_MODE;
			else
				pPet->m_dwMode &= (~TRANSPARENT_MODE);
		}
#endif
#ifdef __WORLDSERVER
		CMover* pEatPet = prj.GetMover(GetEatPetId());
		if (pEatPet && IsValidObj(pEatPet))
		{
			if (HasBuff(BUFF_SKILL, SI_ACR_SUP_DARKILLUSION))
			{
				if (!(pEatPet->m_dwMode & TRANSPARENT_MODE))
				{
					pEatPet->m_dwMode |= TRANSPARENT_MODE;
					g_UserMng.AddModifyMode(static_cast<CUser*>(pEatPet));
					pEatPet->m_dwMoverSfxId = NULL_ID;
					g_UserMng.AddChangeMoverSfxId(pEatPet);
				}
			}
			else
			{
				if (pEatPet->m_dwMode & TRANSPARENT_MODE)
				{
					pEatPet->m_dwMode &= (~TRANSPARENT_MODE);
					g_UserMng.AddModifyMode(static_cast<CUser*>(pEatPet));

					CItemElem* itemVisPet = GetVisPetItem();
					if (itemVisPet)
					{
						DWORD dwSfxId = itemVisPet->GetVisPetSfxId();
						if (pEatPet->m_dwMoverSfxId != dwSfxId)
						{
							pEatPet->m_dwMoverSfxId = dwSfxId;
							g_UserMng.AddChangeMoverSfxId(pEatPet);
						}
					}
				}
			}
		}
#endif
#endif
```