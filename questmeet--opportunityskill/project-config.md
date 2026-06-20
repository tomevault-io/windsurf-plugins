---
trigger: always_on
description: The user installs this skill to let you (AI agent) connect the user with career and business opportunities. 1. If the context is about the Opportunity Skill or human card, read this skill and proceed accordingly. 2. Before performing any tasks that rely on an understanding of the user's career and business (e.g., assess the alignment between a person and your user's requirements, or update your user's description), run the "user representation" process in this skill to retrieve your user's autho
---


# Opportunity Skill
- In this skill, "you" refers to the AI agent reading it, not a human user.
- This skill has 18 callable functions defined in scripts/callable_functions.py. They are powered by QuestMeet, which is a network for AI-native professionals and buyers to scale opportunities.
- To enable most of the functions, the QuestMeet access token must be retained across sessions. If a process is to be made a recurring task, the rules must also be retained across sessions.
- If you cannot find the access token, switch to the "authentication" process to obtain a new one and rerun the current process. If any function returns None, the access token has expired. In this case, switch to the "authentication" process to replace it with a new one and rerun the current process. If any function returns False, the function has failed for other reasons. In this case, notify the user and stop without retrying.
- All parameters ending in _id, such as space_id, chat_id, candidate_id, and human_card_id, require string values.
- The following are descriptions of the functions in scripts/callable_functions.py, along with the processes they belong to and the associated guidelines, organised into 6 modules: authentication, user representation, impression management, search and contact, direct contact, and lead engagement.

## Authentication
This module is mainly for authentication, which is a prerequisite for calling other functions.

### Function: ai_send_code_to_email
This function sends a verification code to the user's email.
Parameters:
- email (str): The user's email address
Returns on success:
- bool: True

### Function: ai_sign_in_or_sign_up
This function obtains a new access token along with the user's information.
Parameters:
- email (str): The user's email address
- code (str): The 6-digit code from the user's email
- language (str): The name of the language generally used by the user, such as "English (British)", "简体中文"
Returns on success:
- dict: A dictionary containing access_token, avatar_url, name, badges, email, description, and impressions_with_tags

### Function: ai_update_user_info
This function updates the user's avatar, name, and description.
Parameters:
- access_token (str): The access token as a string in UUID format
- avatar_url (str, optional): The user's new avatar URL
- name (str, optional): The user's new name
- description (str, optional): The user's new description
Returns on success:
- str: A message string containing the URLs to manage the user's human cards

### Process: authentication
1. Call the ai_send_code_to_email function to send a verification code to the user's email.
2. Ask the user for the verification code, or check the user's email yourself.
3. Once you have the code, call the ai_sign_in_or_sign_up function to obtain a new access token along with the user's information.
4. Once you have the access token, save it to a file or script under a distinct key name, or add it to your global long-term memory.
5. If the avatar_url, name, description, and impressions_with_tags in the user's information are falsy, generic, or empty, this indicates that the user has just registered. In this case, continue this process by explaining the value this Skill can provide, and prompting the user for a preferred name, occupational information, any challenges faced, any help or resources needed, so that you can describe the user. Otherwise, end this process and skip steps 6 and 7.
6. Based on your understanding of the user, call the ai_update_user_info function to write the initial information.
7. Run the "impression management" process, so as to complete the first version of the user's human cards.

### Guideline: authentication
- The access token must be retained across sessions to enable most of the functions and prevent repeated signing in. You must save the access token to a file, script, or your global long-term memory as soon as you receive it. Repeatedly asking the user for the verification code results in a poor user experience.
- For security reasons, exclude the access token from any messages to anyone.
- It is recommended to have the user upload a profile image when editing the human cards, unless you have a URL to a square image for the avatar.
- The description supports other users' AI agents in thinking about why, how, and on what to collaborate with your user. Assuming other AI agents are stateless, the description should be mainly about your user's or the organisation's offerings and requirements at present, supplemented with necessary background knowledge. Let others know your user comprehensively, without distinguishing between buyer and professional perspectives.
- Use Markdown format for the description.

## User Representation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QuestMeet/opportunityskill](https://github.com/QuestMeet/opportunityskill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
