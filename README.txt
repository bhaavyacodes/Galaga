# GALAGA README

## Overview
This C program implements a simplified single-obstacle version of the classic 1981 arcade game *Galaga*, 
where players control a spaceship at the bottom of a 3-lane track to dodge and shoot descending enemies. 
The game runs in a Windows console using ASCII art for visuals, real-time keyboard input, collision detection, 
scoring, and high score persistence via a text file. Background music and sound effects enhance the retro arcade feel.

## Controls
- **Arrow Keys**: Move left/right (lanes 0-2), up/down (within visible track rows 1 to 12)
- **Space/Z**: Shoot a bullet upward (limited to one active bullet)
Controls use non-blocking input via `kbhit()` and `_getch()` from `conio.h` for responsive gameplay.

## Gameplay Mechanics
- **Player**: Represented as `(O)` in the player's lane and row.
- **Obstacle**: Single enemy `V` spawns randomly in a lane at row 0 and descends.
- **Bullet**: `^` fires upward from player position; hitting obstacle awards 5 points and respawns enemy.
- **Scoring**: 1 point for dodging (enemy passes bottom), 5 for shooting; high score saved to `highscore.txt`.
- **Collision**: Game over if player position matches obstacle.
Game resets on 'Y' after game over, with looped background music.

## Technical Details
```
Track Layout (3 lanes, 12 visible rows above player):
|    |    |    |  <- Top (row 0, obstacles spawn here)
...
|  ^ |    |    |  <- Bullet example
...
|    |    | (O) |  <- Player at bottom (row 12)
```

- **Game Loop**: 120ms delay per frame; handles input, updates bullet/obstacle/collision, redraws screen.
- **Screen**: Fast cursor reset via Windows API; borders and cells drawn with fixed-width spacing.
- **Audio**: `PlaySound()` from `mmsystem.h` for bg.wav (loop), shoot.wav, impact.wav, over.wav (requires .wav files in executable directory).
- **High Score**: Loaded/saved from `highscore.txt` using standard file I/O.
Visuals use simple ASCII: player `(O)`, obstacle `V`, bullet `^`.

## Requirements & Setup
- **Compiler**: Windows-compatible (MinGW/GCC/Dev-C++/Visual Studio).
- **Libraries**: Link `winmm.lib` for audio (`#pragma comment(lib, "winmm.lib")`).
- **Audio Files**: Place `bg.wav`, `shoot.wav`, `impact.wav`, `over.wav` in the same folder as executable.
- **Console**: Run in Windows Command Prompt; sets red background (`system("color 4F")`).
**Non-portable**: Only on Windows

## Compilation & Run
```
gcc -o galaga.exe galaga.c -lwinmm
./galaga.exe
```
Create .wav files or comment out `PlaySound()` calls for silent testing. High score persists across runs.

## Limitations & Improvements
- Single enemy (not waves/formations like original Galaga).
- No multiple lives, power-ups, or enemy bullets.
- Windows-only.
- Add: Multiple obstacles, multiple lives, enemy dive patterns, dual-ship mechanic.