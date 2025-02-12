# DiscordSDK Flyff Implementation
_This document will help applying the discord SDK to your flyff server.
There are x86 and x64bit implementations avaliable for those that havent
separated bins in different folders via delayed loading. The implementation
may be updated with more features._

_This also includes a fix with Continent.cpp as I use this to determine map string. This fix can extend to fixing the actual map system and CheckLoad system_

## Patch notes
---
[4/26/2021]
_-- CContinent::GetInstance()->getWorld() only gets called once now rather than twice in setLocationString
-- ContinentDef.h edits has been added_

---
## Features
- Discord Activity implementation
This includes "Time started", "character name and level", and Location by default.
- Option based implementation
Allows users to display the information they want to display rather forcefully.
- Site URL listed.

## Planned Features
---
- Discord Friends list in game
- Party joining through discord api
- (TS is better for this) proximity voip or main channels
- Inviting people on discord.
- Discord bot implementation with app.


## Required Files
---

| File | Download |
| ------ | ------ |
| DiscordSDK | https://drive.google.com/file/d/1aQBU2Ryg2nxbcJlOh8DOVevpeVZSo1Ig/view?usp=sharing |
| FlyFF Image Package | https://drive.google.com/file/d/1DVPd5YBWpXZ_epxiGBRvDE-EQVHeTV2n/view?usp=sharing |
| Flyff Source | Redacted Website |
| Discord Flyff Implementation | Included below
- You will need the SDK in order to implement discord as it is the bare backbones.
- *FlyFF Image Package* Contains default location images and class images. This being the small and large image. 
when there is no class image found jobnpc is used.

## Installation
---
1. Extract the discordSDK to a folder that is nearby or in your source directory. For instance, for my personal projects I prefer having a "Libraries" directory for all the external libraries like Lua, FMod, Miles Sound Systems
2. [files now included] //Extract the contents of **Discord FlyFF Implementation** into the source's Neuz folder. This is only a Neuz implementation currently.
3. Right click on the Neuz Project in Visual Studio and add a Filter labeleld as discord and create a subfolder labeled similarly or discordSDK.
4. [This is where you want to add the disc.cpp and .h file that is below] //Next, add the files you just added to the Neuz folder do the main discord filter in Visual Studio.
5. Rename the CPP folder from the DiscordSDK folder to "discord". Then add every file inside of that folder to the subfilter we would have created "discordSDK" in Visual Studio. 
6. Right click the Neuz Project and open the property pages.
    - Under C/C++, additional includes and edit. Add the location for the discordSDK. Ie: "..\..\Libraries\DiscordSDK\"
    - After, click Linker and then Additional Library Directories. Depending on which version add "..\..\Libraries\DiscordSDK\lib\x86_64" (This folder is 64 bit. 32bit is the x86 folder).
    - [64bit only] -> Go to Input under Linker and add "discord_game_sdk.dll" to "DelayLoadedDlls"
7. (web) Go to https://discord.com/developers/applications and Login.
    - Create a new application.
    - In General Information tab, copy the "Application ID" and replace the define "CLIENT_ID" with that number in disc.h. This is to grab the correct information for the application.
    - Afterwards, select the Rich Prescence tab and the Assets tab. Add a cover image and add all the images in the Image pack.
    - If you want a custom main image, just swap out "main" for your main image.
8. Drag and drop the desired DLL from the DiscordSDK/Lib folder to your client and add it to patch if server is already up. 
    - The folder 64BitClientSide is if you have a 64bit implementation. Otherwise, use the DLL in x86

------

## Adding parts to the source
   - **dpclient.cpp**  
   
   
        **CDPClient::OnSetExperience**  
	_when level is set we need to change the users level_
        ```CPP
        #ifdef __Discord
	    	    const unsigned long oldLevel = pMover->GetLevel();
        #endif
		        pMover->SetExperience( nExp1, (int)wLevel );
        #ifdef __Discord
		        if (oldLevel != static_cast<unsigned long>(wLevel) && pMover == g_pPlayer)
			        g_disc.updateCharacterName();
        #endif
        ``` 
       find: &nbsp;&nbsp;**CDPClient::OnSetChangeJob**  
        _when job is set we need to change the users job_
        ```CPP
		if( pMover->IsActiveMover() )
		{
        #ifdef __Discord
			g_disc.updateCharacterName();
        #endif
        ```
       find: &nbsp;&nbsp;**CDPClient::OnGameJoin**  
       _when the user gets connected, we apply the character information_
        ```CPP
            g_WndMng.PutString( strMessage, NULL, 0xff009900 );
        #ifdef __Discord
    	   g_disc.updateCharacterName();
        #endif
        ```
       <br/> 
   -  Neuz.cpp  
       find: &nbsp;&nbsp;**CNeuzApp::FrameMove()**
         ```CPP
    	    g_dpCertified.ReceiveMessage();
        #ifdef __Discord
	        g_disc.heartbeat();
        #endif
        ```
       <br/> 
   -  Stdafx.cpp
         ```CPP
        #ifdef __Discord
            disc g_disc;
        #endif
        ```
       <br/> 
   -  Stdafx.h
        ```CPP
        #ifdef __Discord
            #include "disc.h"
            extern disc g_disc;
        #endif
        ```
       <br/> 
   -  HwOption.h  
    _These are the options that get added to let the user display what they want to display. More options could be added for instance "Don't show location but show general location through the image on discord"_
        ```CPP
        #ifdef __Discord
        	bool discShowLocation = true;
        	bool discShowCharacter = true;
	        bool discShowTime = true;
        #endif
        ```
       <br/> 
   -  HwOption.cpp  
    in: &nbsp;&nbsp;***COption::Load***
        ```CPP
        #ifdef __Discord
	        else if (scan.Token == _T("discShowLocation"))
	        	discShowLocation = scan.GetNumber() != 0;
	        else if (scan.Token == _T("discShowCharacter"))
	        	discShowCharacter = scan.GetNumber() != 0;
	        else if (scan.Token == _T("discShowTime"))
	        {
	        	discShowTime = scan.GetNumber() != 0;
	        	g_disc.showTimer(discShowTime);
	        }
        #endif
        ```
       <br/>in: &nbsp;&nbsp;***COption::Save***
        ```CPP
        #ifdef __Discord
	        _ftprintf(fp, _T("discShowLocation %d\n"), static_cast<int>(discShowLocation));
	        _ftprintf(fp, _T("discShowCharacter %d\n"), static_cast<int>(discShowCharacter));
	        _ftprintf(fp, _T("discShowTime %d\n"), static_cast<int>(discShowTime));
        #endif
        ```
   -  WndManager.cpp  
           find: &nbsp;&nbsp;**CWndMgr::OpenTitle( BOOL bFirstTime )**  
	   _Every single frame we check the heartbeat and check / do any changes / callbacks_
        ```CPP
        #ifdef __Discord
	        	g_disc.updateCharacterName();
		        g_disc.setLocationString("", 0);
        #endif  
        }
      ```
   -  Wndworld.cpp  
    find: &nbsp;&nbsp;**CWndWorld::Process**  
    _Using the navigator settings we will change the discord name locations. There are two edits in the same function._
        ```CPP
            CWndNavigator* pWndNavigator = (CWndNavigator*)g_WndMng.GetWndBase( APP_NAVIGATOR );
			if (pWndNavigator && lpRegionElem->m_szTitle[0] == 0)
			{
			    pWndNavigator->SetRegionName("");
        #ifdef __Discord
				g_disc.setLocationString("", pWorld->GetID());
        #endif
			}
        ```
        ```CPP
        	CWndNavigator* pWndNavigator = (CWndNavigator*)g_WndMng.GetWndBase( APP_NAVIGATOR );
			if( pWndNavigator )
				pWndNavigator->SetRegionName( szDesc );
		  	if( ::GetLanguage() != LANG_JAP )
			  	g_Caption1.AddCaption( szDesc, m_pFontAPITitle );// CWndBase::m_Theme.m_pFontCaption );
			else
				g_Caption1.AddCaption( szDesc, NULL );// CWndBase::m_Theme.m_pFontCaption );
        #ifdef __Discord
				g_disc.setLocationString(szDesc, pWorld->GetID());
        #endif
        ```
   -  [64bit only] Delay Load of dll's. : In NeuzMsgProc add the following function
        ```CPP
        #ifdef __Discord
        #include <delayimp.h>
        #pragma comment(lib, "delayimp")

        FARPROC WINAPI delayHook(unsigned dliNotify, PDelayLoadInfo pdli)
        {
	        if (pdli)
        	{
	        	if (dliNotify == dliNotePreLoadLibrary)
		        {
        			if (_strcmpi(pdli->szDll, "discord_game_sdk.dll") == 0)
        #if _WIN64
    				return reinterpret_cast<FARPROC>(LoadLibrary("discord_game_sdk_x64.dll"));
        #else
		    		return reinterpret_cast<FARPROC>(LoadLibrary("discord_game_sdk_x86.dll"));
        #endif
		            return reinterpret_cast<FARPROC>(LoadLibrary(pdli->szDll));
		       }
	        }
	        return nullptr;
        }
        ExternC const PfnDliHook __pfnDliNotifyHook2 = delayHook;
        ```
        
   -  Continent.cpp Fixes that are included with discord:
        find and make it this:
        _this makes it so the towns are properly read as it would expect more information when using realData._
        ```CPP
        		else if( script.Token == _T( "C_useRealData" ) )
		        {
			        script.GetNumber( );
	        	}
	   ```
        add functions: 
        ```CPP
        #if defined(__Discord) && defined(__CLIENT)
        unsigned char CContinent::worldIdToMCD(const unsigned long worldId, const bool getWorld)
        {
        	switch (worldId)
        	{
        		case WI_WORLD_MADRIGAL: return getWorld ? CONT_ALL : CONT_FLARIS;
        		case WI_WORLD_KEBARAS: return CASHAREA_ASRIA;
        		case WI_WORLD_CISLAND: return CASHAREA_CORALICELAND;
        		case WI_WORLD_RARTESIA: return CASHAREA_RARTESIA;
        		case WI_WORLD_DARKRARTESIA: return CASHAREA_DARKRARTESIA;
        		case WI_WORLD_HEAVEN01: return DUNGEON_WdHeaven01;
        		case WI_WORLD_HEAVEN02: return DUNGEON_WdHeaven02;
        		case WI_WORLD_HEAVEN03: return DUNGEON_WdHeaven03;
        		case WI_WORLD_HEAVEN04: return DUNGEON_WdHeaven04;
        		case WI_WORLD_HEAVEN05: return DUNGEON_WdHeaven05;
        		case WI_INSTANCE_OMINOUS: return DUNGEON_OMINOUS01;
        		case WI_INSTANCE_OMINOUS_1: return DUNGEON_OMINOUS02;
        		case WI_INSTANCE_DREADFULCAVE: return DUNGEON_DREADFULCAVE;
        		case WI_INSTANCE_RUSTIA: return DUNGEON_RUSTIA_1;
        		case WI_INSTANCE_RUSTIA_1: return DUNGEON_RUSTIA_2;
        		case WI_INSTANCE_BEHAMAH: return DUNGEON_BEHEMOTH;
        		case WI_INSTANCE_KALGAS_1: return DUNGEON_KALGAS;
        		case WI_INSTANCE_KALGAS_2: return DUNGEON_KALGAS;
        		case WI_INSTANCE_UPRESIA: return DUNGEON_UPRESIA;
        		case WI_INSTANCE_KALGAS: return DUNGEON_KALGAS;
        		case WI_INSTANCE_HERNEOS: return DUNGEON_HERNEOS;
        		case WI_INSTANCE_SANPRES: return DUNGEON_SANPRES;
        		case WI_INSTANCE_UPRESIA_1: return DUNGEON_UPRESIA_1;
        		case WI_INSTANCE_HERNEOS_1: return DUNGEON_HERNEOS_1;
        		case WI_INSTANCE_SANPRES_1: return DUNGEON_SANPRES_1;
        		case WI_DUNGEON_FL_MAS: return DUNGEON_MASDUNGEON;
        		case WI_DUNGEON_DA_DK: return DUNGEON_DEKANES;
        		case WI_DUNGEON_VOLCANE: return DUNGEON_VOLCANE;
        		case WI_DUNGEON_SA_TA: return DUNGEON_IBLESS;
        		case WI_DUNGEON_SA_TA2: return DUNGEON_IBLESS;
        		case WI_INSTANCE_CONTAMINTRAILS: return DUNGEON_ContaminatedTrails;
        		case WI_WORLD_ARENA: return AREA_ARENA;
        		case WI_WORLD_GUILDWAR:
        		case WI_WORLD_GUILDWAR1TO1_0:
        		case WI_WORLD_GUILDWAR1TO1_1:
        		case WI_WORLD_GUILDWAR1TO1_2:
        		case WI_WORLD_GUILDWAR1TO1_3:
        			return AREA_GUIDLWAR;
        		default: return CONT_NODATA;
        		
        	}
        }
        
        unsigned char CContinent::getWorld(CMover* player)
        {
        	if (!player)
        	{
        		if (g_pPlayer)
        			player = g_pPlayer;
        		else
        			return CONT_NODATA;
        	}
        
        	const CWorld* playerWorld = player->GetWorld();
        	if (playerWorld)
        	{
        		unsigned char location = worldIdToMCD(playerWorld->GetID(), true);
        		if (location != CONT_ALL)
        			return location;
        
        		location = GetTown(player->GetPos());
        		if (location != CONT_NODATA)
        			return location;
        
        		location = GetContinent(player->GetPos());
        		if (location != CONT_NODATA)
        			return location;
        	}
        	return 1;
        }
        
        std::string CContinent::getLocationString(const unsigned char location) 
        {
        	switch (location)
        	{
        		case CONT_FLARIS: return "Flaris";
        		case CONT_SAINTMORNING: return "Saint Morning";
        		case CONT_RICIS: return "Rhisis Garden";
        		case CONT_ESTIA: return "Shaduwar"; //??
        		case CONT_DARKON12: return "Darkon";
        		case CONT_DARKON3: return "Darkon 3";
        		case CONT_HARMONIN: return "Valley of the Risen";
        		case CONT_KAILLUN: return "Kailun";
        		case CONT_BAHARA: return "Bahara Dessert";
        		case TOWN_SAINCITY: return "Saint City";
        		case TOWN_DARKEN: return "Darken City";
        		case TOWN_ELIUN: return "Eliun";
        		case TOWN_FLARINENOSPLE: return "Flarine";		
        		case CASHAREA_ASRIA: return "Azria";
        		case CASHAREA_CORALICELAND: return "Coral Island";
        		case CASHAREA_RARTESIA: return "Rartesia";
        		case CASHAREA_DARKRARTESIA: return "Dark Rartesia";
        		case DUNGEON_WdHeaven01: return "Tower F1";
        		case DUNGEON_WdHeaven02: return "Tower F2";
        		case DUNGEON_WdHeaven03: return "Tower F3";
        		case DUNGEON_WdHeaven04: return "Tower F4";
        		case DUNGEON_WdHeaven05: return "Tower F5";
        		case DUNGEON_OMINOUS01: return "Animus";
        		case DUNGEON_OMINOUS02: return "Curesed Animus";
        		case DUNGEON_DREADFULCAVE: return "Catacombs of Anguish";
        		case DUNGEON_RUSTIA_1: return "Wilds";
        		case DUNGEON_RUSTIA_2: return "Savage Wilds";
        		case DUNGEON_BEHEMOTH: return "Ankou's Asylum";
        		case DUNGEON_KALGAS: return "Kalgas";
        		case DUNGEON_UPRESIA: return "Upresia";
        		case DUNGEON_UPRESIA_1: return "Upresia M";
        		case DUNGEON_HERNEOS: return "Herneos";
        		case DUNGEON_SANPRES: return "Sanpres";
        		case DUNGEON_HERNEOS_1: return "Herneos M";
        		case DUNGEON_SANPRES_1: return "Sanpres M";
        		case DUNGEON_MASDUNGEON: return "Mars Mine";
        		case DUNGEON_DEKANES: return "Dekane Mine";
        		case DUNGEON_VOLCANE: return "Volkane";
        		case DUNGEON_IBLESS: return "Ibliss Dungeon";
        		case DUNGEON_ContaminatedTrails: return "Contaminated Trails";
        		default: return "Madrigal";
        	}
        }
        #endif
        ```
      
   -  Continent.h
      ```CPP
        #if defined(__Discord) && defined(__CLIENT)
	        unsigned char getWorld(CMover* player = nullptr);
	        static unsigned char worldIdToMCD(unsigned long worldId, bool getWorld = false);
	        static std::string getLocationString(unsigned char location);
        #endif
        ```
   -  ContinentDef.h in Resource.  _Defining new areas for the map_
        ```CPP
        #define AREA_ARENA 70
        #define AREA_GUIDLWAR 71
        ```
   -  World.h Changing something to const
        ```CPP
			[[nodiscard]] auto GetId() const { return m_dwWorldID; }
        ```
   -  Make a file named "disc.h" / "disc.cpp" and add the following to it. Afterwards, add the file to the neuz project.
        disc.h
        ```CPP
        #pragma once
        #include "Discord/discord.h"
        #include <memory>
        
        constexpr const char defaultLocationString[] = "GaiaFlyFF.com";
        constexpr const char defaultCharacterString[] = "";
        constexpr const char defaultDiscordIconBig[] = "main";
        constexpr const char defaultDiscordIconSmall[] = "jobnpc";
        constexpr const __int64 CLIENT_ID = 834630024929214474;
        
        struct DiscordState
        {
        	discord::User currentUser;
        	std::unique_ptr<discord::Core> core;
        };
        
        class disc
        {
        		DiscordState state{};
        
        		time_t checkOpen;
        		time_t checkInstalled;
        		time_t hb;
        		const time_t started;
        
        		discord::Activity mainActivity{};
        		std::string currentLocation;
        		std::string currentLocationImage;
        		bool hasChangedActivity;
        	
        
        	public:
        		disc();
        		~disc() = default;
        		disc(disc&&) = delete;
        		disc(const disc&) = delete;
        		disc& operator=(const disc&) = delete;
        		disc& operator=(disc&&) = delete;
        
        		bool createCore();
        		void updateActivity();
        		void heartbeat();
        
        		void showTimer(bool opt = false);
        		void updateCharacterName();
        
        		void setLocationString(std::string_view name, unsigned long id = 0);
        		static std::string getDiscordMapString(unsigned long id); 
        		void toggleEnableLocation();
		};
		#endif
        ```
        
   -  Disc.cpp
        ```CPP
        #include "StdAfx.h"
        #include "disc.h"
        #include "Discord/discord.h"
        #include "Continent.h"
        #pragma comment(lib, "discord_game_sdk.dll.lib")
        
        #ifdef __Discord
        bool disc::createCore() 
        {
        	discord::Core* core{};
        	discord::Result result = discord::Core::Create(CLIENT_ID, DiscordCreateFlags_NoRequireDiscord, &core);
        	switch (result)
        	{
        		case discord::Result::InvalidPermissions:
        		case discord::Result::NotInstalled:
        		case discord::Result::NotFound:
        			checkInstalled = time(nullptr);
        			return false;
        		case discord::Result::InternalError:
        		case discord::Result::NotRunning:
        			checkOpen = time(nullptr);
        			return false;
        		case discord::Result::Ok:
        			state.core.reset(core);
        			if (state.core)
        			{
        				state.core->UserManager().OnCurrentUserUpdate.Connect([this]()
        					{
        						discord::Result result = state.core->UserManager().GetCurrentUser(&state.currentUser);
        						if (result != discord::Result::Ok)
        							Error("Failed getting user %d", static_cast<unsigned int>(result));
        					});
        
        
        				mainActivity.SetType(discord::ActivityType::Playing);
        				mainActivity.SetApplicationId(CLIENT_ID);
        				mainActivity.SetDetails(defaultCharacterString);
        				mainActivity.SetState(defaultLocationString);
        				mainActivity.GetAssets().SetSmallImage(defaultDiscordIconSmall);
        				mainActivity.GetAssets().SetLargeImage(defaultDiscordIconBig);
        				hasChangedActivity = true;
        				
        				state.core->RunCallbacks();
        				hb = time(nullptr) + SEC(15);
        				return true;
        			}
        			return false;
        		default:
        			Error("[Discord]: Err %d", static_cast<unsigned int>(result));
        			return false;
        	}
        }
        
        disc::disc() : checkOpen(0), checkInstalled(0), hb(0), started(time(nullptr)), hasChangedActivity(false)
        {
        	createCore();
        }
        
        
        void disc::heartbeat()
        {
        	const time_t currentTime = time(nullptr);
        	if (checkInstalled != 0)
        	{
        		if (checkInstalled > currentTime)
        		{
        			if (!createCore())
        				checkInstalled = currentTime + MIN(10);
        			else
        				checkInstalled = 0;
        		}
        		return;
        	}
        	if (checkOpen != 0)
        	{
        		if (checkOpen > currentTime)
        		{
        			if (!createCore())
        				checkOpen = currentTime + MIN(5);
        			else
        				checkOpen = 0;
        		}
        		return;
        	}
        
        	if (hb > currentTime)
        	{
        		if (state.core == nullptr)
        		{
        			checkOpen = currentTime + MIN(5);
        			return;
        		}
        		hb = currentTime + SEC(15);
        		if (hasChangedActivity)
        			updateActivity();
        						
        		switch (state.core->RunCallbacks())
        		{
        			case discord::Result::InternalError:
        			case discord::Result::NotRunning:
        				checkOpen = currentTime + MIN(5);
        				break;
        			case discord::Result::InvalidPermissions:
        			case discord::Result::NotInstalled:
        			case discord::Result::NotFound:
        				checkInstalled = currentTime + MIN(10);
        				break;
        			default:
        				break;
        		}			
        	}
        }
        
        void disc::updateCharacterName()
        {
        	if (g_pPlayer && g_Option.discShowCharacter)
        	{
        		std::string tempstr = g_pPlayer->GetName();
        		tempstr += "  |  Lv. ";
        		tempstr += std::to_string(g_pPlayer->GetLevel());
        		switch (g_pPlayer->GetLegendChar())
		        {
			        case LEGEND_CLASS_MASTER: tempstr += "-M"; break;
			        case LEGEND_CLASS_HERO: tempstr += "-H"; break;
			        case LEGEND_CLASS_LEGENDHERO: tempstr += "-H"; break;
			        case LEGEND_CLASS_NORMAL: default:  break;
		        }
        	    mainActivity.SetDetails(tempstr.c_str());

        		std::string classImage = "job";
        		switch (g_pPlayer->m_nJob)
        		{
        			case JOB_VAGRANT: classImage += "vagrant"; break;
        			case JOB_MERCENARY: classImage += "mercenary"; break;
        			case JOB_ACROBAT: classImage += "acrobat"; break;
        			case JOB_MAGICIAN: classImage += "magician"; break;
        			case JOB_ASSIST: classImage += "assist"; break;
        			case JOB_KNIGHT: classImage += "knight"; break;
        			case JOB_BLADE: classImage += "blade"; break;
        			case JOB_JESTER: classImage += "jester"; break;
        			case JOB_RANGER: classImage += "ranger"; break;
        			case JOB_RINGMASTER: classImage += "ringmaster"; break;
        			case JOB_BILLPOSTER: classImage += "billposter"; break;
        			case JOB_PSYCHIKEEPER: classImage += "psychikeeper"; break;
        			case JOB_ELEMENTOR: classImage += "elementor"; break;
        			case JOB_KNIGHT_MASTER: classImage += "knightmaster"; break;
        			case JOB_BLADE_MASTER: classImage += "blademaster"; break;
        			case JOB_JESTER_MASTER: classImage += "jestermaster"; break;
        			case JOB_RANGER_MASTER: classImage += "rangermaster"; break;
        			case JOB_RINGMASTER_MASTER: classImage += "ringmastermaster"; break;
        			case JOB_BILLPOSTER_MASTER: classImage += "billpostermaster"; break;
        			case JOB_PSYCHIKEEPER_MASTER: classImage += "psychikeepermaster"; break;
        			case JOB_ELEMENTOR_MASTER: classImage += "elementormaster"; break;
        			case JOB_KNIGHT_HERO: classImage += "knighthero"; break;
        			case JOB_BLADE_HERO: classImage += "bladehero"; break;
        			case JOB_JESTER_HERO: classImage += "jesterhero"; break;
        			case JOB_RANGER_HERO: classImage += "rangerhero"; break;
        			case JOB_RINGMASTER_HERO: classImage += "ringmasterhero"; break;
        			case JOB_BILLPOSTER_HERO: classImage += "billposterhero"; break;
        			case JOB_PSYCHIKEEPER_HERO: classImage += "psychikeeperhero"; break;
        			case JOB_ELEMENTOR_HERO: classImage += "elementorhero"; break;
        			case JOB_LORDTEMPLER_HERO: classImage += "lordtemplerhero"; break;
        			case JOB_STORMBLADE_HERO: classImage += "stormbladehero"; break;
        			case JOB_WINDLURKER_HERO: classImage += "windlurkerhero"; break;
        			case JOB_CRACKSHOOTER_HERO: classImage += "crackshooterhero"; break;
        			case JOB_FLORIST_HERO: classImage += "floristhero"; break;
        			case JOB_FORCEMASTER_HERO: classImage += "forcemasterhero"; break;
        			case JOB_MENTALIST_HERO: classImage += "mentalisthero"; break;
        			case JOB_ELEMENTORLORD_HERO: classImage += "elementorlordhero"; break;
        			default: classImage += "npc"; break;
        		}
        		mainActivity.GetAssets().SetSmallImage(classImage.c_str());
        	}
        	else
        	{
        		mainActivity.SetDetails(defaultCharacterString);
        		mainActivity.GetAssets().SetSmallImage(defaultDiscordIconSmall);
        	}
        
        	hasChangedActivity = true;
        }
        
        void disc::showTimer(const bool opt)
        {
        	if (opt)
        		mainActivity.GetTimestamps().SetStart(started);
        	else
        		mainActivity.GetTimestamps().SetStart(0);
        	hasChangedActivity = true;
        }
        
        std::string disc::getDiscordMapString(const unsigned long id)
        {
        	switch (id)
        	{
        		case CONT_FLARIS: return "world_flaris";
        		case CONT_SAINTMORNING: return "world_saint";
        		case CONT_RICIS: return "world_ricis";
        		case CONT_ESTIA: return "world_harmonin"; 
        		case CONT_DARKON12: return "world_darkon12";
        		case CONT_DARKON3: return "world_darkon3";
        		case CONT_HARMONIN: return "world_estia";
        		case CONT_KAILLUN: return "world_kaillun";
        		case CONT_BAHARA: return "world_bahara";
        		case TOWN_SAINCITY: return "town_saincity";
        		case TOWN_DARKEN: return "town_darken";
        		case TOWN_ELIUN: return "town_elliun";
        		case TOWN_FLARINENOSPLE: return "town_flarine";
        		case CASHAREA_ASRIA: return "world_Azria";
        		case CASHAREA_CORALICELAND: return "maploading_cisland";
        		case CASHAREA_RARTESIA: return "world_Rartesia";
        		case CASHAREA_DARKRARTESIA: return "world_darkRartesia";
        		case DUNGEON_WdHeaven01: return "dungeon_wdheaven01";
        		case DUNGEON_WdHeaven02: return "dungeon_wdheaven02";
        		case DUNGEON_WdHeaven03: return "dungeon_wdheaven03";
        		case DUNGEON_WdHeaven04: return "dungeon_wdheaven04";
        		case DUNGEON_WdHeaven05: return "dungeon_wdheaven05";		
        		case DUNGEON_OMINOUS02: case DUNGEON_OMINOUS01: return "maploading_ominous";
        		case DUNGEON_DREADFULCAVE: return "maploading_dreadfulcave";
        		case DUNGEON_RUSTIA_2: case DUNGEON_RUSTIA_1: return "maploading_rustia";
        		case DUNGEON_BEHEMOTH: return "maploading_behemoth";
        		case DUNGEON_KALGAS: return "maploading_kalgas";
        		case DUNGEON_UPRESIA_1: case DUNGEON_UPRESIA: return "dungeon_upresia";
        		case DUNGEON_HERNEOS_1: case DUNGEON_HERNEOS: return "dungeon_herneos";
        		case DUNGEON_SANPRES_1: case DUNGEON_SANPRES: return "maploading_flyship";		
        		case DUNGEON_MASDUNGEON: return "dungeon_Masdungeon";
        		case DUNGEON_DEKANES: return "dungeon_dekanes";
        		case DUNGEON_VOLCANE: return "dungeon_volcane";
        		case DUNGEON_IBLESS: return "maploading_ibless";
        		case DUNGEON_ContaminatedTrails: return "maploading_risiscontamin";
        		case AREA_ARENA: return "maploading_arena";
        		case AREA_GUIDLWAR: return "maploading_guildwar";
        		case CONT_NODATA: default:
        			return "world_flyff";
        	}	
        }
        
        void disc::setLocationString(const std::string_view name, const unsigned long id)
        {
        	if (id == 0)
        	{
        		mainActivity.SetState(defaultLocationString);
        		mainActivity.GetAssets().SetLargeImage(defaultDiscordIconBig);
        		return;
        	}
        
        	const unsigned char locationMCD = CContinent::GetInstance()->getWorld();	
        	if (name.empty()) 
        		currentLocation = CContinent::getLocationString(locationMCD);
        	else
        		currentLocation = name;
        
        	currentLocationImage = getDiscordMapString(locationMCD);
        	toggleEnableLocation();
        }
        
        void disc::toggleEnableLocation()
        {
        	if (g_Option.discShowLocation)
        	{
        		mainActivity.SetState(currentLocation.c_str());
        		mainActivity.GetAssets().SetLargeImage(currentLocationImage.c_str());
        	}
        	else
        	{
        		mainActivity.SetState(defaultLocationString);
        		mainActivity.GetAssets().SetLargeImage(defaultDiscordIconBig);
        	}
        	hasChangedActivity = true;
        }
        
        void disc::updateActivity()
        {
        	state.core->ActivityManager().UpdateActivity(mainActivity, [](discord::Result result) {
        		if (result != discord::Result::Ok)
        		{
        			Error("discord result bad activitiy or sumthing uwu %d", static_cast<int>(result));
        		}
        		});
        	hasChangedActivity = false;
        }
        #endif
        ```

## Older Compiler Edits
---
_For users too lazy to get to C++17 / newest_

1. Changing out std::string_view
    - the function "setLocationString" needs to be altered. You can alter it to using const char* like the following. Now this function itself would have less overhead compared to the std::string_view.
        ```CPP
        void setLocationString(const char* name, unsigned long id = 0);
        ```
        ```CPP
        void disc::setLocationString(const char * name, const unsigned long id)
        {
	        if (id == 0)
        	{
        		mainActivity.SetState(defaultLocationString);
        		mainActivity.GetAssets().SetLargeImage(defaultDiscordIconBig);
        		return;
        	}
        
        	const unsigned char locationMCD = CContinent::GetInstance()->getWorld();
        	if (name == nullptr || strlen(name) == 0)
        		currentLocation = CContinent::getLocationString(locationMCD);
        	else
        		currentLocation = name;
        
        	currentLocationImage = getDiscordMapString(locationMCD);
        	toggleEnableLocation();
        }
        ```
        
## Sylistic Choices
---
- Disc.h contains a few default variables that you can change:
    - defaultCharacterString (top string and the default string to show when the option is disabled)
    - defaultLocationString (bottom string. Also the default string when location disabled)
    - defaultDiscordIconBig (The bigger icon used in discord.)
    - defaultDiscordIconSmall (the smaller icon. set to "" if you dont want a small image)
    - CLIENT_ID -> This is the variable you set to your application id.
- If there are any stylistic changes that need to be had, I will assist on making them happen for a while after purchasing. 

## Setting up Discord Application
---
_In order to use discord integrartion, you must set up a discord application through the discord developer portal. The reason behind this is I am not liable for keeping the test one up and running or in the same state._

1. [Discord Developer Portal](https://discord.com/developers/applications), Login and create new application.
2. Here you may set the application name, the default icon for future uses and you can grab the application id.
3. on the left hand side, there are a few options. Go to Art Assets and then upload the images you want or the image pack. Beware, the images will be set to lower case -- discord uses lowercase for the keys.
4. If there is a different style choice you may want, you can always mess around with the visualizer. 

## Thanks
---
_FlyFF is an mmorpg created by GalaNet/Aeonsoft. This implementation was done by myself:_
| Account  | Site |
| -------- | ------- |
| Kiaemi. | Discord |
| Velvet | RageZone |
| M¿dScientist | Elitepvpers |



Please also follow the discord rules for having an application: [Discord Legal](https://discord.com/developers/docs/legal)