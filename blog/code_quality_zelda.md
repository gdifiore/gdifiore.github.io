# A Retrospective on Code Quality

This post details some thoughts I had when looking over some code I wrote for a [university assignment](https://github.com/CSE3902-Group-3/CSE3902-TheLegendOfZelda-Remake) (a remake of the original NES Legend of Zelda).

I won't be pushing any of these changes, but I thought it would be a good test of what I have learned, and an exercise in how I would approach refactoring code I haven't seen in a while (or ever).

## Too many variables

The `Link` class ended up needing to manage 3 different positions:

1. the position of the current sprite (used for drawing)
2. the position of the collider (used to resolve colisions)
3. the position stored in the state machine (used for movement)

My solution was to write the following utility function:

```c#
// LinkUtilities.cs
public static void UpdatePositions(Link link, Vector2 newPositon)
{
    link.Sprite.UpdatePos(newPositon);
    link.StateMachine.position = newPositon;
    link.Collider.Pos = newPositon;
}
```

It's a fine solution, I guess, but it's not in line with the other calling conventions of `Link`-related variables. Consistency makes it easier for others to pick up your code. Cases like this are when it's useful to understand design patterns that the language you're using implements.

Enter, the **Property Pattern**

```c#
// Link.cs
private Vector2 position;

public Vector2 Position
{
    get => position;
    set
    {
        position = value;
        Sprite.UpdatePos(position);
        StateMachine.position = position;
        Collider.Pos = position;
    }
}
```

Yes, this introduces one more variable to a semi-complicated class, but it makes updating the position way more consitent with other code.

Maybe a better refactor would to make all classes reference the `LinkStateMachine` position instead? But if you did not write/maintain another other class that uses a different position? That would introduce errors and/or bugs.

## Unifying responsibilities
```c#
// Modified LinkCollisionWithItem.cs
namespace LegendOfZelda
{
    public class LinkCollisionWithItem
    {
        public static void HandleCollisionWithItem(CollisionInfo collision)
        {
            IItem itemCollidedWith = collision.CollidedWith.Collidable as IItem;

            if (itemCollidedWith == null)
                return;

            switch (itemCollidedWith)
            {
                case Bow:
                case Triforce:
                    Inventory.getInstance().AddItem(itemCollidedWith);
                    GameState.Link.StateMachine.ChangeState(new CollectItemLinkState(itemCollidedWith));
                    break;
                ...
            }

            // Play sound only for specific items
            if (!(itemCollidedWith is Heart || itemCollidedWith is OneRupee || itemCollidedWith is FiveRupee))
            {
                SoundFactory.PlaySound(SoundFactory.getInstance().GetItem);
            }
        }
    }
}
```

Why handle a bunch of `if-not` cases at the bottom? If `Heart`, `OneRupee`, and `FiveRupee` all handle sound on pickup, so should the other items!

This is where a concrete design before starting would be useful.

## Trying to get fancy

`Utility/Config/ReadConfig.cs` contains a wrapper for reading from the config INI file that can be optionally edited.

```ini
; config.ini
[Link]
Speed=5
Health=6
ProjectileSpawnCooldown=1.5

[Enemies]
; Easy damage multiplier = 0.5, Medium = 1; Hard = 2
Difficulty=Medium

[Game]
Level=1
```

I wanted to avoid calling every function individually:

```c#
GetLinkHealth();
GetLinkSpeed();
GetLinkProjectileSpawnCooldown();
GetDifficulty();
GetStartLevel();
```

but in trying to be fancy with anonymous functions, I ended up creating something uglier:

```c#
// ReadConfig.cs
...
        private delegate void anonFunc();
        private anonFunc[] anonFuncs = new anonFunc[numConfig];

        ...
        public ReadConfig(string iniFilePath)
        {
            ...
            anonFuncs[0] = GetLinkSpeed;
            anonFuncs[1] = GetLinkHealth;
            anonFuncs[2] = GetLinkProjectileSpawnCooldown;
            anonFuncs[3] = GetDifficulty;
            anonFuncs[4] = GetStartLevel;

            foreach (var func in anonFuncs)
            {
                func?.Invoke();
            }
...
```

Why use anonymous functions here? I am doing almost the exact same thing I wanted to avoid.

All but one of those `ini` entries directly reads a float value, so why not create a generic function create a map of variables to the value from the ini?
*Difficulty would require a special function because the `ini` expects a string that gets mapped to a damage multiplier.*

```c#
// Modified ReadConfig.cs
...
        // Mapping of config keys to the methods that handle them
        private Dictionary<string, Action> configHandlers;

        public ReadConfig(string iniFilePath)
        {
            ...
            configHandlers = new Dictionary<string, Action>
            {
                { "Link.Speed", () => GetConfigValue("Link", "Speed", "Link.Speed") },
                { "Link.Health", () => GetConfigValue("Link", "Health", "Link.Health") },
                { "Link.ProjectileSpawnCooldown", () => GetConfigValue("Link", "ProjectileSpawnCooldown", "Link.ProjectileSpawnCooldown") },
                { "Game.Difficulty", GetDifficulty },
                { "Game.Level", () => GetConfigValue("Game", "Level", "Game.Level") }
            };

            // Execute each config handler
            foreach (var handler in configHandlers.Values)
            {
                handler.Invoke();
            }
            ...
        }

        ...

        private void GetConfigValue(string section, string key, string configKey)
        {
            ...
        }
...
```
Much cleaner. New config values can be added in one line, rather than adding an entire new function. This is also more dynamic if the config file was to be changed to `JSON`, `GetConfigValue(...)`'s internals can be modified, but leaving the signature alone.

## What causes bad designs/implementations?

1. **Time restraints** <br>
This assignment was (among other things) designed for our groups to use sprint-planning. So there was always a quick turn around between tasks, as well as bug-fixes, tesing of other PRs, code reviews, etc. it was easy to just say "Oh this works, I'll just push it to a PR".
15 (really, 13) weeks is a quick turnaround for a project like this. Also, add on all the other classes, hobbies, and a social life and time starts to feel limited.

2. **Lack of Knowledge** <br>
This was the first large assignment that I had to start from scratch (albeit, with a group) and that's very challenging the first time. The ability to pick the right design patterns, separate concerns, implement proper error handling are all things that were tested. And I certainly failed at first on some of those, but have learned and have a new outlook on design and implementation.

## Lessons learned

#### Design, design, design
Even if it takes 5 hours, designing and drawing relationships between clases and responsibilities can help identify potential complexity before you start coding.

You don't need to write complete architecture or design documents, but map what's in your mind onto paper.

## Conclusion
I think reviewing old code is a good idea for any software developer. You get to see if you've learned new skills or gained a new perspective on design. Everyone struggles to read old code, even if they wrote it. Exercise your brain, do things you think are challenging.

<hr>
<b><center>Me on: <a href="https://github.com/gdifiore/">GitHub</a>
