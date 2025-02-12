## Monster HP Bars above Model
- CWndWorld::OnEraseBkgnd
  ```cpp
	if( pFocus && pFocus->GetType() == OT_MOVER )
	{
		if( ((CMover *)pFocus)->IsMode( TRANSPARENT_MODE ) )		// »ó´ë°¡ Åõ¸íÈ­ »óÅÂ¸é
			pWorld->SetObjFocus(nullptr );							// Å¸°ÙÀâÀº°Å Ç®¸².
  #ifdef __MonsterHPBars
		else
		{
			if (!dynamic_cast<CMover*>(pFocus)->IsPeaceful())
			{
				// In this case, I only implemented the dot and normalize function -- copied from the party members HP as you can tell from "v3PartyMemberDir"
				v3PartyMemberDir = dynamic_cast<CMover*>(pFocus)->GetPos() - g_Neuz.m_camera.m_vPos;
				D3DXVec3Normalize(&v3PartyMemberDir, &v3PartyMemberDir);

				if (D3DXVec3Dot(&v3CameraDir, &v3PartyMemberDir) >= 0.0f)
					dynamic_cast<CMover*>(pFocus)->RenderHP(g_Neuz.m_pd3dDevice);
			}
		}
  #endif
  ```

> Note: If you have this system, double check to see if it was the original released system from ages ago. That one would be missing the D3DXVec3Dot calculation causing the hp bar to render while the monster is offscreen.