## Kills for Guild Contribution
----
System for Simple flyff.

- Project.h
```cpp
#ifdef __GUILD_KILL_CONTRI
#include <unordered_map>
class GuildContrKills
{
	public:
		struct LevelKillInfo {
			unsigned short Start_Level;
			unsigned long Points;
		};

	private:
		bool System_Loaded;
		std::unordered_map<unsigned long, unsigned long> Monster_Point_List;
		std::vector<LevelKillInfo> Level_Point_List;

	public:
		GuildContrKills() : System_Loaded(false) {}
		void Load();

		[[nodiscard]] bool IsLoaded() const noexcept { return System_Loaded; }
		[[nodiscard]] unsigned long GetPoint(unsigned long Monster_ID, unsigned short Monster_Level) const;
};

#endif
```

then in the class CProject

```cpp
#ifdef __GUILD_KILL_CONTRI
	GuildContrKills gck;
#endif
```


- Project.cpp
<small>CProject::OpenProject</small>

```cpp
#ifdef __GUILD_KILL_CONTRI
	gck.Load();
#endif
```

```cpp
#ifdef __GUILD_KILL_CONTRI
#include <unordered_map>
void GuildContrKills::Load()
{
	Level_Point_List.push_back({ 0, 1 });

	CScript s;
	if (!s.Load("GuildContriKill.txt"))
	{
		System_Loaded = false;
		return;
	}

	s.GetToken();
	while (s.tok != FINISHED)
	{
		if (s.Token == "MONSTER")
		{
			const unsigned long Monster_Or_Level = s.GetNumber();
			const unsigned long Kill_Reward = s.GetNumber();
			Monster_Point_List.insert_or_assign(Monster_Or_Level, Kill_Reward);
		}
		else if (s.Token == "LVL")
		{
			const unsigned long Monster_Or_Level = s.GetNumber();
			const unsigned long Kill_Reward = s.GetNumber();
			Level_Point_List.push_back({static_cast<unsigned short>(Monster_Or_Level), Kill_Reward});			
		}
		s.GetToken();
	}
	if (!Level_Point_List.empty())
		std::sort(Level_Point_List.begin(), Level_Point_List.end(), [](const LevelKillInfo& a, const LevelKillInfo& b) { return b.Start_Level > a.Start_Level; });

	System_Loaded = true;
}

unsigned long GuildContrKills::GetPoint(const unsigned long Monster_ID, const unsigned short Monster_Level) const
{
	if (const auto iter = Monster_Point_List.find(Monster_ID); iter != Monster_Point_List.end())
		return iter->second;

	unsigned long Points_Value = 0;
	for (const auto& [Start_Level, Points] : Level_Point_List)
	{
		if (Monster_Level >= Start_Level && Points_Value < Points)
			Points_Value = Points;
	}
	return Points_Value;
}
#endif
```

- AttackArbiter.cpp
```cpp
#ifdef __GUILD_KILL_CONTRI
#include "DPCoreClient.h"
extern CDPCoreClient g_DPCoreClient;
#endif
```

```cpp
#ifdef __GUILD_KILL_CONTRI
	if (m_pAttacker->GetGuild() && m_pAttacker->IsPlayer())
	{
		if (const auto* mProp = m_pDefender->GetProp(); m_pDefender->IsNPC() && mProp)
			g_DPCoreClient.Contribute(dynamic_cast<CUser*>(m_pAttacker), prj.gck.GetPoint(mProp->dwID, mProp->dwLevel), 0);
	}
#endif

	m_pAttacker->AddKillRecovery();

```



## License
---
_FlyFF is an mmorpg created by GalaNet/Aeonsoft. This implementation was done by myself:_
| Account  | Site |
| -------- | ------- |
| Kiaemi. | Discord |
| Velvet | RageZone |
| M¿dScientist | Elitepvpers |

