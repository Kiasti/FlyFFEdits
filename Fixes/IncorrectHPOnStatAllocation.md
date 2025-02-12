
## Fix with HP and MP being set when setting a stat point
- **CDPClient::OnSetState**
	_OnSetState is when you set stat points_
	```cpp
	g_pPlayer->SetHitPoint( g_pPlayer->GetMaxHitPoint() );
	g_pPlayer->SetManaPoint( g_pPlayer->GetMaxManaPoint() );
	```