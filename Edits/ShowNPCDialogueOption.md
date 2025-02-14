# Disable NPC Dialogue Option  
_The chat bubbles can get annoying for the player and this has been quite well recieved._

## Changelog
---
- Feb 13, 2025 - Implemented ClearNpcMessages

---
## Adding parts to the source
---
   - **HWOption.h**  
	_Somewhere in class COption but under public._
     ```CPP
		bool ignoreNpcDialogue = false;
     ```   	
   - **HWOption.cpp**  
	_Add to the Load function under another option._
     ```CPP
		else if (scan.Token == _T("IgnoreNpcDialogue"))
			ignoreNpcDialogue = scan.GetNumber() != 0;
     ```
     _Add to the Save function under another option._
     ```CPP
		_ftprintf(fp, _T("IgnoreNpcDialogue %d\n"), static_cast<int>(ignoreNpcDialogue));
     ```   	
   - **DPClient.cpp (Neuz)**  
	in: CDPClient::OnChat
     ```CPP
		if (strlen(szBuf) && (szBuf[0] == '/' || szBuf[0] == '!'))
		{
			if (pMover->IsNPC() && g_Option.ignoreNpcDialogue)
				return;
			g_DialogMsg.AddMessage(pMover, szBuf);
		}
		else
		{
			if (pMover->IsNPC() && g_Option.ignoreNpcDialogue)
				return;
			sprintf_s(szChat, "%s : %s", pMover->GetName(true), szBuf);
			g_WndMng.PutString(szChat, pMover, 0xffffffff, CHATSTY_GENERAL);
		}
     ```   	


---

Upon setting the option, remove the existing npc dialogues.

```cpp
void CDialogMsg::ClearNpcMessages(const bool MonsterMessages)
{
	for (int i = 0; i < m_textArray.GetSize(); i++)
	{
		const tagCUSTOMTEXT* lpCustomText = static_cast<tagCUSTOMTEXT*>(m_textArray.GetAt(i));
		if (auto* object = lpCustomText->m_pObj; IsValidObj(object) && object->GetType() == OT_MOVER)
		{
			if (static_cast<CMover*>(object)->IsNPC() && (static_cast<CMover*>(object)->IsPeaceful() || MonsterMessages))
			{
				SAFE_DELETE(lpCustomText);
				m_textArray.RemoveAt(i);
				i--;
			}
		}
	}
}
```

Whenever you change the option via your option window:
```cpp
g_DialogMsg.ClearNpcMessages(false);
```





## Thank you
---
_FlyFF is an mmorpg created by GalaNet/Aeonsoft. This implementation was done by myself:_
| Account  | Site |
| -------- | ------- |
| Kiaemi. | Discord |
| Velvet | RageZone |
| M¿dScientist | Elitepvpers |

