
## Making Non-Elemental Skills Not Get A Bonus On Non-Elemental Item
- **MoverAttack.cpp** : CMover::GetMagicSkillFactor
  ```CPP
  	if (skillType == itemType && skillType != SAI79::NO_PROP)
  ```
_There's a lot of skills that are non-elemental based. When it's a magic skill, it double checks if the item element is the same as the skill element to give it increased damage -- or to reduce damage when it isn't. Because of this, when elementing a wand, your multiplier goes from 1.1 to 1.0 or 0.9. Ideally, you just remove the check for when the Skill has no element, unless you want to handle more (E.g. making the non-element damage, elemental damage based on your item element)_

