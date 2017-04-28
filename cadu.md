# Cadu

Cadu, from Caduceus, the symbol of Hermes.

## Intent

To assist people with mundane tasks who might otherwise not be able to afford an assistant.

Partly: Builds a schedule for a user to be comfortable and flexible for them, especially as a person with ADHD.

## Parties involved

1. Administrator _(The website operator)_
2. Moderator _(The assistant)_
3. User _(The client)_

## Features

### Client side

- Login functionality
- Scheduler/Calendar
- Messaging
- Task requests
    - backside: see request, accept it
    - save requests
- Notification system (when task is accepted, when task is done, that you have a reminder waiting, etc.)
- Reminder system (separate from notifications): reminders are manual from the assistant (either requested from the client/scheduled or noticed by the assistant)
- Reward system (for the user): maybe sign-ins, tasks completed
- General task list (for the user) that assistant can pick things off of like a wishlist
- Ability to print stuff/export documents or whatever
- "Quick memo" - take quick reminder to add to list later, because you don't remember what you don't remember
- Subscription payment functionality
- Preferences functionality

### Assistant side

- Login functionality
- Ability to generate reports based on repeated data (Hey Jeff, I noticed you miss this particular task every Monday...)
- Dashboard
    - Mailbox
    - Task scheduler, listed by client
        - not yet accepted, accepted, in progress, completed - status bar/icon next to each
        - Task object, assign to someone (could be client or assistant), gets accepted (auto for client)
- Notification system for upcoming scheduled tasks
- Separate scheduler for 15min - 1h consultations
- Payment functionality

### Administrator

- Payment stuff
- Ability to make someone an Assistant

## User stories

- blah

## Miscellaneous notes

- If an assistant changes something (e.g. a schedule), the client gets notified, and must accept that change explicitly
    - There should be preferences, so you can shut off notifications/confirmations if you'd like

## Code and structure

### Models

- User
- Task
- Notification
- Reminder
- Preference
- Message

#### User

- name
- email
- password_digest
- assistant_id (optional, if assistant)
- role
    - client
    - assistant
    - admin

#### Task

May add tags to tasks... partly because it's good to be able to categorize tasks into types (requires phone call, requires travel, etc.) and partly because my project needs to have a nested form which writes to a model through a custom attribute writer and I can't picture another part of this app that would require that logic.

Have common tasks built in so you can provide info (e.g. doctor's appointment gives you boxes for reason, date, office, etc.).

drop down calendar

warning for requesting a task same day

- content
- timestamp
- date due
- client_id
- status
    - idle
    - requested
    - accepted
    - in progress
    - completed

#### Label

A label is used to denote a type of work required by a task (requires phone call, requires travel, requires foreign language, requires internet access, etc.) and can be created by Assistants, and applied by both Clients and Assistants.

- name

#### LabelsTasks

- label_id
- task_id

#### Notification

- content
- timestamp
- task_id
- status
    - unseen
    - seen
    - dismissed

#### Reminder

- content
- timestamp
- client_id
- status
    - unseen
    - seen
    - completed

#### <s>Message

- subject
- content
- timestamp
- priority
    - low
    - medium
    - high
- sender_id
- receiver_id
- thread_id
- task_id (optional - message can be related to a task)</s>

#### Comment

Going to use a comment thread on a task to discuss information. That way, communication is focused, as opposed to a general message/email system and the Assistant having to deal with organizing info from unattached sources. With comments, all communication is centralized on a topic, and all information necessary to complete a task can be accessed from one area.

- content
- task_id
- author_id
- timestamp
- pinned?
- edited?

#### Preference

- client_id
- phone # & personal info
- _(I'll do these later)_

### To do

1. <s>I've done the models first</s>
2. <s>Build controllers and specific views for each of the functions, with the intent to build dashboards for client and assistant _after_</s>
3. <s>Policies, probably, _after_ (?) the controllers but before the dashboard?</s>
4. <s>'Mark complete' button on tasks (even for clients)</s>
5. <s>'Remind about this' button on tasks (even for clients) - ability to create reminders for yourself as a client, too.</s>
6. <s>"Repeat reminder" for assistant instead of "Dismiss"</s>
7. "Mark seen" button on reminders for clients
7. add labels to things with button/dropdown
8. <s>add new labels</s>
9. 'action' actions moved to smart #update
10. <s>minutes dropdown gets pushed to next line on smaller screens</s>
11. <s>actually use policy scopes for scoping model collections</s>
12. <s>new labels should apply using find_by so it doesn't error</s>
13. **tasks should not have duplicate labels**
14. <s>links to see label index, add new labels (for assistant)</s>
15. **display errors for forms**
16. color overdue tasks or otherwise emphasize them (notification)
16. **allow oauth from some other service like google or facebook**
17. radio buttons on user edit pages for role change (only if you're an administrator)