# Damage Render Format Options  
_Expands the Damage Render option to accept different values to determine separating numbers via comma, space, period, or not at all in the neuz.ini file. (Option window must be made yourselves)._

---
## Adding parts to the source
---
   - **2dRender.cpp**  
     Find: **CDamageNum::Render**
	 
	 ```CPP	
	 #if defined(__DamageRenderComma)
		const int spacePosition = nLength % 3;
		for (int i = 0; i < nLength; ++i)
		{
			if (const unsigned long nIndex = strTemp[i] - '0' + m_nAttribute * 12; static_cast<int>(nIndex) >= 0)
			{
				if (i % 3 == spacePosition && i)
				{
					switch (g_Option.m_bDamageRender)
					{
						case 3:
							textPackNum.Render(&g_Neuz.m_2DRender, CPoint(static_cast<int>(fX), static_cast<int>(fY)), m_nAttribute * 16 + 11, static_cast<DWORD>(nAlpha), fScaleX, fScaleY);

							fX = static_cast<FLOAT>(fX + (textPackNum.m_ap2DTexture[m_nAttribute * 12 + 11].m_size.cx * 0.5 * fScaleX));
							break;
						case 2:
							textPackNum.Render(&g_Neuz.m_2DRender, CPoint(static_cast<int>(fX), static_cast<int>(fY)), m_nAttribute * 16 + 10, static_cast<DWORD>(nAlpha), fScaleX, fScaleY);
							fX = static_cast<FLOAT>(fX + (textPackNum.m_ap2DTexture[m_nAttribute * 12 + 10].m_size.cx * 0.5 * fScaleX));
							break;
						case 1:
							fX = static_cast<FLOAT>(fX + (textPackNum.m_ap2DTexture[nIndex].m_size.cx * 0.5 * fScaleX));
							break;
						case 0:
						default:
							break;

					}
				}
				textPackNum.Render(&g_Neuz.m_2DRender, CPoint(static_cast<int>(fX), static_cast<int>(fY)), nIndex, static_cast<DWORD>(nAlpha), fScaleX, fScaleY);
				fX = static_cast<FLOAT>(fX + (textPackNum.m_ap2DTexture[nIndex].m_size.cx * 0.5 * fScaleX));
			}
		}
	 #else
		for (int i = 0; i < nLength; i++)
		{
			DWORD nIndex = strTemp[i] - '0' + m_nAttribute * 14;
			if( (int)nIndex >= 0 )	// ¿¡·¯¹æÁö.
			{
				textPackNum->Render( &g_Neuz.m_2DRender, CPoint( (int)( fX ), (int)( fY ) ), nIndex ,(DWORD)nAlpha,fScaleX,fScaleY);
			} 
			else
			{
				Error( "DamageNum::Render : %s %d", strTemp, m_nAttribute );
			}
			
			fX	= (FLOAT)( fX + (textPackNum->m_ap2DTexture[nIndex].m_size.cx*0.5*fScaleX) );
		}
	 #endif
     ```
## Thank you
---
_FlyFF is owned and operated and developed by Galanet/Aeonsoft. This is only an implementation made by myself, Kia#1411._

[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen. Thanks SO - http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)

