## Cheer Increasing Upgrade Rates Idea
_It's always been a rumor but it was never a thing. To this day, you'll see people ask for cheer, and most servers even don't have an implementation like this._

- Example from SmeltAccessory
  ```CPP
	unsigned long dwProbability = CAccessoryProperty::GetInstance()->GetProbability( pItemMain->GetAbilityOption() );
	if (pUser->HasBuff(BUFF_ITEM, II_CHEERUP))
	{
		if (const int temp = static_cast<int>(dwProbability * cheerValueUpgrade); temp < 100) 
			dwProbability += 100; 
		else 
			dwProbability += temp; 
	}
  ```

>Note: Only one function is provided. If you want to add the feature, you'll need to add it to all the Safe upgrades and non-safe upgrades, as they use different functions.