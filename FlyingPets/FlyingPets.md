# Flying Pets  
_Forces all EGG and PET types to be flying. This also makes them follow the player while the player is on a flying device without forcing the user to put the pet away._

---
## Adding parts to the source
---
   - **ActionMoverCollision.cpp**  
	in: **CActionMover::ProcessCollisionFly**
     ```CPP
		int nAttr = pWorld->GetHeightAttribute(pPos->x, pPos->z);

	 #ifdef __FlyingPets
		MoverProp* const mProp = m_pMover->GetProp();
		const auto isPet = mProp ? mProp->dwAI == AII_PET || mProp->dwAI == AII_EGG : false;
		if (!isPet)
	 #endif
			switch (nAttr)
			{
				case HATTR_NOMOVE:
					pPos->x -= vDelta.x;
					break;
				default: break;
			}

		pPos->z += vDelta.z;
		pWorld->ClipZ(pPos->z);
		if (pWorld->m_bFly)
			nAttr = pWorld->GetHeightAttribute(pPos->x, pPos->z);
	 #ifdef __FlyingPets
		else if (!isPet)
	 #else
		else
	 #endif
			nAttr = HATTR_NOFLY;

	 #ifdef __FlyingPets
		if (!isPet)
	 #endif
			switch (nAttr)
			{
				case HATTR_NOMOVE:
					pPos->z -= vDelta.z;
					break;
				case HATTR_DIE:
					pMover->DoDie(nullptr);
					break;
				case HATTR_NOFLY:
					pPos->x -= vDelta.x;
					pPos->z -= vDelta.z;
					pPos->y -= vDelta.y;
					pMover->UnequipRide();
					break;
				default: break;
			}

		m_fCurrentHeight = pWorld->GetFullHeight(D3DXVECTOR3(pPos->x, pPos->y + 1.0f, pPos->z));

	 #ifdef __FlyingPets // those who dont want to increase max height to fit landscape
		if (pPos->y > pWorld->m_fMaxHeight && !isPet)
	 #else
		if (pPos->y > pWorld->m_fMaxHeight)
	 #endif
     ```   	
   - **ActionMoverMsg2.cpp**  
	Find: **CActionMover::ProcessActMsg2**
     ```CPP
				if( pItemProp )
					ret = pMover->SetMotion( pItemProp->dwUseMotion + MTA_FSTAND1, ANILOOP_LOOP, MOP_FIXED );		// ¾Ö´Ï¸ÞÀÌ¼Ç ¼³Á¤
		 #ifdef __FlyingPets
				else if (pMover->GetProp()->dwAI == AII_EGG || pMover->GetProp()->dwAI == AII_PET)
					ret = pMover->SetMotion(MTI_STAND, ANILOOP_LOOP, MOP_FIXED);		// ¾Ö´Ï¸ÞÀÌ¼Ç ¼³Á¤
		 #endif
				if( ret == TRUE )
					if( pMover->m_pRide )	pMover->m_pRide->m_fFrameCurrent	= 0;
				break;
			}
		case OBJMSG_STOP:		
		case OBJMSG_ASTOP:
		#ifdef __Y_INTERFACE_VER3
			if( (GetMoveState() == OBJSTA_FMOVE) || (GetMoveState() == OBJSTA_BMOVE) || (GetMoveState() == OBJSTA_LMOVE) || (GetMoveState() == OBJSTA_RMOVE) )	// Àü/ÈÄÁøÁßÀÏ¶§ Á¦ÀÚ¸®¿¡ ¼¼¿î´Ù.
		#else //__Y_INTERFACE_VER3
			if( (GetMoveState() == OBJSTA_FMOVE) || (GetMoveState() == OBJSTA_BMOVE) )	// Àü/ÈÄÁøÁßÀÏ¶§ Á¦ÀÚ¸®¿¡ ¼¼¿î´Ù.
		#endif //__Y_INTERFACE_VER3
			{
				pMover->ClearDest();				// ¸ñÇ¥ ÁÂÇ¥ Å¬¸®¾î.
				SendActMsg( OBJMSG_ACC_STOP );		// °¡¼Ó ÁßÁö
				ResetState( OBJSTA_MOVE_ALL );		
			}
			break;
			// ÀüÁøÇØ¶ó!
		case OBJMSG_FORWARD:
			if( GetMoveState() == OBJSTA_FMOVE )	return 0;// ÀÌ¹Ì ÀüÁø»óÅÂ¸é Ãß°¡·Î Ã³¸® ÇÏÁö ¾ÊÀ½
			SetMoveState( OBJSTA_FMOVE );		// 
			if( IsActTurn() )	break;
			if( GetState() & OBJSTA_ATK_ALL )	
				return 1;
				
		 #ifdef __FlyingPets
			if (pMover->GetProp()->dwAI == AII_EGG || pMover->GetProp()->dwAI == AII_PET)
				pMover->SetMotion(MTI_WALK, ANILOOP_LOOP, MOP_FIXED);	// ÀÏ´ÜÀº ´Ù ´ë±â µ¿ÀÛÀ¸·Î ¾´´Ù
			else
		 #endif
			if( !pItemProp )
			{
				Error( "ItemProp is NULL in ProcessActMsg2()\n" );
				return 0;
			}		
			pMover->SetMotion( pItemProp->dwUseMotion + MTA_FRUNNING1, ANILOOP_LOOP, MOP_FIXED );	// ÀÏ´ÜÀº ´Ù ´ë±â µ¿ÀÛÀ¸·Î ¾´´Ù
			break;
		// ÁÂ/¿ì ÅÏ ÇØ¶ó! 
		case OBJMSG_LTURN:
			{
			FLOAT fTurnAngle = (FLOAT) nParam1 / 100.0f;
			if( GetTurnState() == OBJSTA_LTURN && m_fTurnAngle == fTurnAngle )	return 0;
			m_fTurnAngle = fTurnAngle;
			SetTurnState( OBJSTA_LTURN );
			if( GetState() & OBJSTA_ATK_ALL )	return 0;
			
		 #ifdef __FlyingPets
				if (pMover->GetProp()->dwAI == AII_EGG || pMover->GetProp()->dwAI == AII_PET)
				{
					if( fTurnAngle > 1.0f )
						pMover->SetMotion(MTI_WALK, ANILOOP_LOOP, MOP_FIXED);	// ÀÏ´ÜÀº ´Ù ´ë±â µ¿ÀÛÀ¸·Î ¾´´Ù		
				}
				else 
		 #endif
				if(!pItemProp )
				{
					Error( "ItemProp is NULL in ProcessActMsg2()\n" );
					return 0;
				}
			if( fTurnAngle > 1.0f )
				pMover->SetMotion( pItemProp->dwUseMotion + MTA_FLTURN1, ANILOOP_LOOP, MOP_FIXED );	

			}
			break;
		case OBJMSG_RTURN:
			{
			FLOAT fTurnAngle = (FLOAT) nParam1 / 100.0f;
			if( GetTurnState() == OBJSTA_RTURN && m_fTurnAngle == fTurnAngle )	return 0;
			m_fTurnAngle = fTurnAngle;
			SetTurnState( OBJSTA_RTURN );
			if( GetState() & OBJSTA_ATK_ALL )	return 0;	// °ø°ÝÁß¿£ È¸Àü¸ð¼Ç ÇÏÁö ¾ÊÀ½.
		 #ifdef __FlyingPets
				if (pMover->GetProp()->dwAI == AII_EGG || pMover->GetProp()->dwAI == AII_PET)
				{
					if( fTurnAngle > 1.0f )
						pMover->SetMotion(MTI_WALK, ANILOOP_LOOP, MOP_FIXED);	// ÀÏ´ÜÀº ´Ù ´ë±â µ¿ÀÛÀ¸·Î ¾´´Ù		
				}
				else
		 #endif 
				if(!pItemProp )
				{
					Error( "ItemProp is NULL in ProcessActMsg2()\n" );
					return 0;
				}
				if( fTurnAngle > 1.0f )
					pMover->SetMotion( pItemProp->dwUseMotion + MTA_FRTURN1, ANILOOP_LOOP, MOP_FIXED );
			}
			break;
		case OBJMSG_STOP_TURN:
			if( GetTurnState() == 0 )		return 0;		// ÀÌ¹Ì ÅÏ »óÅÂ°¡ ¾ø´Ù¸é Ã³¸® ¾ÈÇÔ
			m_fTurnAngle = 0.0f;
			SetTurnState( 0 );		// ÅÏÀ» ÁßÁö
			if( GetState() & OBJSTA_ATK_ALL )	return 0;
		 #ifdef __FlyingPets
				if (pMover->GetProp()->dwAI == AII_EGG || pMover->GetProp()->dwAI == AII_PET)
				{
					if (GetMoveState() == OBJSTA_FMOVE)
						pMover->SetMotion(MTI_WALK, ANILOOP_LOOP, MOP_FIXED);	// ÀÏ´ÜÀº ´Ù ´ë±â µ¿ÀÛÀ¸·Î ¾´´Ù
					else
						pMover->SetMotion(MTI_STAND, ANILOOP_LOOP, MOP_FIXED);
			
				}
				else 
		 #endif
				if(!pItemProp )
     ```
   - **ActionMoverState2.cpp**  
	Find: **CActionMover::ProcessFlyMove**

     ```CPP
	 #ifdef __CLIENT
	 #ifdef __FlyingPets
 		if (MoverProp* moverProp = pMover->GetProp(); moverProp && moverProp->dwAI != AII_PET && moverProp->dwAI != AII_EGG)
	 #endif
			 ProcessFlyParticle(fLenSq);
	 #endif
     ```  
     <br/>Find: **CActionMover::ProcessState2**
     ```CPP
	 #ifdef __FlyingPets
		 const MoverProp* mProp = pMover->GetProp();
	 #endif
		 switch( dwState )
	 	 {
		 // Á¦ÀÚ¸® ´ë±â / Á¤Áö
		 case OBJSTA_STAND:
			m_fAccPower = 0;		// ´ë±â/Á¤Áö»óÅÂ¿¡¼± ÈûÀ» ´õÀÌ»ó °¡ÇÏÁö ¾Ê´Â´Ù.
			if( GetState() & OBJSTA_ATK_ALL )	break;
			if( GetState() & OBJSTA_TURN_ALL )	break;
			if( GetState() & OBJSTA_DMG_ALL )	break;
			{

	 #ifdef __FlyingPets
				if (mProp->dwAI == AII_EGG || mProp->dwAI == AII_PET)
				{
					pMover->SetMotion(MTI_STAND, ANILOOP_LOOP, MOP_FIXED);
					break;
				}
	 #endif
				if( pMover->SetMotion( pItemProp->dwUseMotion + MTA_FSTAND1, ANILOOP_LOOP, MOP_FIXED ) == TRUE )		// ´ë±â»óÅÂ
					if( pMover->m_pRide )	
						pMover->m_pRide->m_fFrameCurrent = 0;
			}
			break;
		// ÀüÁø
		case OBJSTA_FMOVE:
			// ¹«¹ö°¡ Å¸°íÀÖ´Â ¾ÆÀÌÅÛÀÇ ÀÎµ¦½º¿¡¼­ ÃßÁø·ÂÀ» ²¨³»¿È.
	 #ifdef __FlyingPets
			if (mProp->dwAI == AII_EGG || mProp->dwAI == AII_PET)
			{
				m_fAccPower = 0.0038;
				pMover->SetMotion(MTI_WALK, ANILOOP_LOOP, MOP_FIXED);
				break;
			}
	 #endif
			m_fAccPower = pItemProp->fFlightSpeed * 0.75f;

			if( GetState() & OBJSTA_ATK_ALL )	break;
			if( GetState() & OBJSTA_TURN_ALL )	break;
			{
				if( pMover->SetMotion( pItemProp->dwUseMotion + MTA_FRUNNING1, ANILOOP_LOOP, MOP_FIXED ) == TRUE )		// ´ë±â»óÅÂ
					if( pMover->m_pRide )		
						pMover->m_pRide->m_fFrameCurrent = 0;
			}
			break;
		// ÁÂ/¿ì µ¹±â
		case OBJSTA_LTURN:
			{
				float fTurnAng = m_fTurnAngle;
				if( (GetStateFlag() & OBJSTAF_ACC) == 0 )		// °¡¼Ó»óÅÂ°¡ ¾Æ´Ò¶§´Â 2¹è·Î »¡¸® µ·´Ù.
					fTurnAng *= 2.5f;
				pMover->AddAngle( fTurnAng );
			}
			if( IsActAttack() )	break;
	 #ifdef __FlyingPets

			if (mProp->dwAI == AII_EGG || mProp->dwAI == AII_PET)
			{
				pMover->SetMotion(MTI_WALK, ANILOOP_LOOP, MOP_FIXED);
				break;
			}
	 #endif
			pMover->SetMotion( pItemProp->dwUseMotion + MTA_FLTURN1, ANILOOP_LOOP, MOP_FIXED );
		
			break;
		case OBJSTA_RTURN:
			{
				float fTurnAng = m_fTurnAngle;
				if( (GetStateFlag() & OBJSTAF_ACC) == 0 )		// °¡¼Ó»óÅÂ°¡ ¾Æ´Ò¶§´Â 2¹è·Î »¡¸® µ·´Ù.
					fTurnAng *= 2.5f;
				pMover->AddAngle( -fTurnAng );
			}
			if( IsActAttack() )	break;
	 #ifdef __FlyingPets
			if (mProp->dwAI == AII_EGG || mProp->dwAI == AII_PET)
			{
				pMover->SetMotion(MTI_WALK, ANILOOP_LOOP, MOP_FIXED);
				break;
			}
	 #endif
			pMover->SetMotion( pItemProp->dwUseMotion + MTA_FRTURN1, ANILOOP_LOOP, MOP_FIXED );
			break;
     ```
     <br/>
   - **Mover.cpp**  
	Find: **CMover::Process**
     ```CPP
	 #ifdef __CLIENT
		if( IsNPC() && IsFlyingNPC() )				// ºñÇà¸÷Àº ÆÄÆ¼Å¬ »ý¼º
	 #ifdef __FlyingPets
		{
			MoverProp* moverProp = GetProp();
			if (moverProp && moverProp->dwAI != AII_PET && moverProp->dwAI != AII_EGG)
				CreateFlyParticle(this, GetAngleX(), 0);
		}
	 #else
			CreateFlyParticle(this, GetAngleX(), 0);
	 #endif
     ```
     find: **CMover::IsItemRedyTime**
     ```CPP
		 #ifndef __FlyingPets
		 #if __VER >= 15 // __IMPROVE_SYSTEM_VER15
			if( pItemProp->dwItemKind2 == IK2_RIDING )
			{
				if( HasActivatedEatPet() || HasActivatedSystemPet() || HasPet() )
				{
					( (CUser*)this )->AddDefinedText( TID_GAME_CANNOT_FLY_WITH_PET );
					return FALSE;
				}
			}
		 #endif // __IMPROVE_SYSTEM_VER15
		 #endif
     ```
     find: **CMover::IsEquipAble**
     ```CPP
	 #ifndef __FlyingPets
	 #if __VER >= 9	// __PET_0410
	 #ifdef __WORLDSERVER
			if( HasActivatedEatPet() || HasActivatedSystemPet() )	// ÆêÀÌ ¼ÒÈ¯µÈ »óÅÂ¶ó¸é ºñÇà ºÒ°¡
			{
				( (CUser*)this )->AddDefinedText( TID_GAME_CANNOT_FLY_WITH_PET );
				return FALSE;
			}
	 #endif	// __WORLDSERVER
	 #endif	// __PET_0410
	 #endif
     ```
   - **MoverRender.cpp**  
	Find: **CMover::Render**
     ```CPP
	 #ifdef __FlyingPets
			if (MoverProp* mProp = GetProp(); mProp->bKillable == 1 && mProp->dwFlying == 1 && mProp->dwAI != AII_EGG && mProp->dwAI != AII_PET)
	 #else
			if (GetProp()->bKillable == 1 && GetProp()->dwFlying == 1)		// Á×ÀÌ´Â°Ô °¡´ÉÇÑ³Ñ / ºñÇà¸÷ ¸¸ Å¸°ÙÀ¸·Î ÀâÈù´Ù. 
	 #endif
     ```
   - **MoverSkill.cpp**  
	Find: **CMover::ActivateSystemPet**
     ```CPP
	 #ifndef __FlyingPets
		if (IsFly())
		{
			static_cast<CUser*>(this)->AddDefinedText(TID_GAME_CANNOT_CALL_PET_ON_FLYING);
			return;
		}
	 #endif
     ```
     find: **CMover::ActivateEatPet**
     ```CPP
	 #ifndef __FlyingPets
		if (IsFly())
		{
			static_cast<CUser*>(this)->AddDefinedText(TID_GAME_CANNOT_CALL_PET_ON_FLYING);
			return;
		}
	 #endif
     ```
   - **Pet.cpp**  
	Find: **CAIEgg::MoveProcessIdle**
     ```CPP	
	 #ifndef __FlyingPets
			vPos.y = pOwner->GetWorld()->GetLandHeight(pOwner->GetPos());
	 #endif
     ```
     ```CPP
	 #ifndef __FlyingPets
			vDist.y = 0;
	 #endif
			FLOAT fDistSq = D3DXVec3LengthSq(&vDist);
	 #ifdef __FlyingPets
			if (fDistSq > 1.5f) // wait till distance before move
	 #else
			if (fDistSq >1.0f)
	 #endif
     ```
   - **ProjectCmn.cpp**  
	Find: **CProject::LoadPropMover**
     ```CPP	
			pProperty->dwFlying				= scanner.GetNumber();
	 #ifdef __FlyingPets
			if (pProperty->dwAI == AII_EGG || pProperty->dwAI == AII_PET)
				pProperty->dwFlying = 1;
	 #endif
     ```
     
## Thank you
---
_FlyFF is owned and operated and developed by Galanet/Aeonsoft. This is only an implementation made by myself, Kia#1411._

[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen. Thanks SO - http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)

