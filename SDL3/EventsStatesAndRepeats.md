# Events, States, and Key Repeats

When dealing with the buttons of joysticks, mice, and keyboards, it's common that people reach first for events, but SDL also provides, and it's common for people to keep, ways to query state. It's important to understand the differences here, as you may run into undesirable results if you don't choose wisely.

Conceptually, Events are a way for the OS or SDL to inform you something has happened. You don't necessarily _always_ want to react to this information immediately. Consider when the player is moving forward, moving the character is, in most games, going to require per-frame work. So frame to frame, you'll need to know the user is pressing the button. Button events, regardless if they're Keyboard/Mouse/Gamepad/Joystick, don't get sent every frame, although they do get repeated, which we'll discuss down below. This can make it appear as though a character is continuously moving, but it will be jerky and there will be a big delay towards the beginning.

Of course, there's still cases where you might want to use events, but they're typically singular actions, or for adjusting persistant state in lieu of computing it every frame with states. You can know when someone has pressed down on a button to try to do an action, and then you can choose to either immediately adjust state, or set a flag to evaluate the input later in the frame.

On the other hand States, in the context of input, represent what we last saw when we finished processing events. These are great when we may receive contradictory input and we want to evaluate all relevant state before proceeding with your game logic. They're also useful for continuous input, like movement. 

We'll demonstrate both methods below.

### Key Repeat 

As mentioned above, key events are not sent every frame. There's no real way for SDL to do this for us if you're using something like [`SDL_PollEvent`](SDL_PollEvent), as it has no idea when your frame starts and ends. Despite that, we do get repeated key event periodically. These repeats are sent by the OS, and typically there's two types of delays between these events. The first type is for the first repeat event. It's typically a little longer, something like ~1s, and then the OS will start sending them at some interval, something like ~200ms. These numbers are fuzzy as this is generally user configurable and dependent on the OS.

Also mentioned above is that using events for something like movement often results in jerky movement, and indeed, this is due to the delays discussed above. You can mimic the jerky cadence by putting a cursor into a simple text editor and holding it down. The intervals you see as the characters appear is what the key repeat looks like on your system.

## States

For states you check during the gameloop, you can retrieve the state of the keyboard via [`SDL_GetKeyboardState`](SDL_GetKeyboardState). This will give you an array of boolean values representing if a button is up (`false`), or down (`true`), indexed by [`SDL_Scancodes`](SDL_Scancode). There's no need to worry about the lifetime, SDL owns this array, but you may want to cache it in practice, just so you don't have to call into SDL all the time.

With states, you can recompute a movement direction every frame, rather than keeping it persistant and modifying it as events come in.

```c
typedef int2 {
    int x, y;
} int2;

int2 direction_user_should_move()
{
    const bool *key_states = SDL_GetKeyboardState();
    int2 direction;

    if (key_states[SDL_SCANCODE_W]) {
        direction->y += 1;  /* pressed what would be "W" on a US QWERTY keyboard. */
    } 
    
    if (key_states[SDL_SCANCODE_S]) {
        direction->y -= 1;  /* pressed what would be "S" on a US QWERTY keyboard. */
    } 
    
    if (key_states[SDL_SCANCODE_A]) {
        direction->x -= 1;  /* pressed what would be "A" on a US QWERTY keyboard. */
    } 
    
    if (key_states[SDL_SCANCODE_D]) {
        direction->x += 1;  /* pressed what would be "D" on a US QWERTY keyboard. */
    }

    return direction;  /* wasn't key in W or S location, don't move. */
}
```

## Events

On the other hand, you might want to keep a persistent direction state, in that case you can modify it as events come in. Simply grab the `scancode` field from [SDL_EVENT_KEY_DOWN](SDL_EVENT_KEY_DOWN) and [SDL_EVENT_KEY_UP](SDL_EVENT_KEY_UP) events.

```c
typedef int2 {
    int x, y;
} int2;

void direction_user_should_move(const SDL_Event *e, int2 *direction)
{
    SDL_assert(e->type == SDL_EVENT_KEY_DOWN || e->type == SDL_EVENT_KEY_UP); /* just checking key presses/releases here... */

    /* If we're pressing a button, we want to add movement in that direction */
    int adjustment = 1;

    /* If we're releasing a button, we want to subtract movement in that direction */
    if (e->type == SDL_EVENT_KEY_UP) {
        adjustment = -1;
    }

    if (e->key.scancode == SDL_SCANCODE_W) {
        direction->y += adjustment;  /* pressed what would be "W" on a US QWERTY keyboard. */
    } else if (e->key.scancode == SDL_SCANCODE_S) {
        direction->y -= adjustment;  /* pressed what would be "S" on a US QWERTY keyboard. */
    } else if (e->key.scancode == SDL_SCANCODE_A) {
        direction->x -= adjustment;  /* pressed what would be "A" on a US QWERTY keyboard. */
    } else if (e->key.scancode == SDL_SCANCODE_D) {
        direction->x += adjustment;  /* pressed what would be "D" on a US QWERTY keyboard. */
    }
}
```

## Seeing it in practice

The [snake](demo/01-snake) and [woodeneye](02-woodeneye-008) demos both demonstrate something similar to the event method above for it's movement. It may be instructive to take a look at their source and test them out.
