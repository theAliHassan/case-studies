# Free My TV

A fast, ad-free custom home screen for a slow Google TV.

**Built on:** [simbasjungles/free-my-tv](https://github.com/simbasjungles/free-my-tv) by Simba and contributors (MIT). **Status:** v1.0 complete (October 2026). **Code:** private, because the fork carries settings and customizations for my own TV.

## What was theirs and what was mine

The upstream project provides the launcher, the switch between Free My TV and the stock home, the ad blocking and the base of the settings, plus the LG and Samsung web apps. My fork redesigned the home experience on top of that. Of the 328 commits in the repository, 97 are mine (23–26 October 2026); the rest are the upstream project's.

What I built on top:

- A paged home (Home, Apps, TV and an optional Tools page) over a single wallpaper, replacing the old home and its row editors.
- A Tools page with weather, local time and prayer times, and a second clock.
- An Environment Canada weather alert (Canada only).
- A wallpaper screen with 45 photographs, day and night sets, and a screensaver that shares the home's layout.
- A "Free Up Memory" action, screen-reader (TalkBack) support, and 24 language catalogues.
- A visible focus style designed to be read from across the room.

## The problem

The target was a TCL Google TV with 1.4 GB of memory, Android 11 and a 720p interface. It was slow and cluttered with promotions. The first design constraint I wrote down was speed: moving focus on the stock-style home missed its frame deadline 66% of the time on that TV (median frame time 26 ms).

## Approach

1. **Measure first.** I built small shell tools that drive the TV over `adb` and record frame timing and memory, so that every change could be compared on the real device, not only on an emulator.
2. **Find the real cause.** The old home drew everything into a few large layers that were redrawn on every focus move, which is what made the TV jank. The new home is made of real child views, so a focus or page change only changes a view's scale, transparency or position, which the GPU handles without redrawing. Later memory work traced one large memory cost to a focus shadow in the settings screen.
3. **Draw by hand, ship no libraries.** The app is plain Java, drawn on a canvas, with no third-party libraries and no background services of its own. Nothing animates while nobody is watching.
4. **Keep it reversible.** A one-step "Original" option brings back the factory home screen, and a watchdog does the same automatically after repeated crashes. The app needs no root.
5. **Regress visually.** Each release was compared against a gallery of screenshots (54 shots on the last run, plus a worst-case layout test at the largest text size).

## Results

Measured by me on one TV with the project's own scripts (build 1.0, versionCode 52; the numbers are in the project notes):

| | Before (stock-style home) | Free My TV |
|---|---|---|
| Frames missing their deadline while moving focus | 66% | 0.62% (24 focus moves) |
| Frames missing their deadline on page changes | n/a | 0.14% (32 page changes) |
| Memory of the home screen | 51 MB | 39.9 MB at rest |
| Frames drawn while idle | n/a | 0 |
| Screensaver process memory | n/a | 26 MB |

"Free Up Memory" returned 68 MB on the TV in the last test.

## Limits

- One TV model was measured. Other Google TVs need adapting, and I wrote a porting guide for it, but I have not tested them.
- Samsung (Tizen) and LG (webOS) TVs are not supported by this fork.
- **Known layout issue.** At the largest text size on a 720p screen, a strip on the Tools page can sit very close to, and in a worst-case date, cover part of the date. I confirmed it with a test and left it unfixed as a layout choice.
- The numbers above are my own measurements on one device, so treat them as indicative, not as a benchmark.
- The wallpapers belong to their photographers and are not included.

## What I would change

- Add automated checks for the layout logic, which today is covered only by the screenshot gallery and manual runs.
- Test on a second, different TV before calling the porting guide proven.
- Publish a clean public copy. The build tooling for that exists, but I decided to keep the fork private for now.
