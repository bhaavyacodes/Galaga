#include <stdio.h>
#include <conio.h>
#include <windows.h>
#include <stdlib.h>
#include <time.h>
#include <mmsystem.h>
#pragma comment(lib, "winmm.lib")

/* ---------- CONFIG ---------- */

#define LANES 3
#define TRACK_HEIGHT 12            // visible rows above player
#define MAX_BULLETS 3
#define HIGHSCORE_FILE "highscore.txt"

/* ---------- GLOBALS ---------- */

int playerLane = 1;                // 0..2 (left/mid/right)
int playerRow  = TRACK_HEIGHT;     // vertical position (row index)
int obstacleLane;
int obstacleRow;
int bulletActive = 0;
int bulletLane;
int bulletRow;
int score = 0;
int highScore = 0;
int gameOver = 0;

/* ---------- UTILS ---------- */

void clear_screen_fast() {
    HANDLE h = GetStdHandle(STD_OUTPUT_HANDLE);
    COORD pos = {0, 0};
    SetConsoleCursorPosition(h, pos);
}

/* Simple delay */
void sleep_ms(int ms) {
    Sleep(ms);
}

/* ---------- HIGHSCORE HANDLING ---------- */

void loadHighScore() {
    FILE *f = fopen(HIGHSCORE_FILE, "r");
    if (f == NULL) {
        highScore = 0;
        return;
    }
    if (fscanf(f, "%d", &highScore) != 1) {
        highScore = 0;
    }
    fclose(f);
}

void saveHighScore() {
    FILE *f = fopen(HIGHSCORE_FILE, "w");
    if (f == NULL) {
        return;
    }
    fprintf(f, "%d\n", highScore);
    fclose(f);
}

/* ---------- GAME OBJECTS ---------- */

void resetObstacle() {
    obstacleLane = rand() % LANES;
    obstacleRow  = 0;
}

/* Reset player, obstacle, bullet, score for a fresh game */
void resetGame() {
    playerLane = 1;
    playerRow  = TRACK_HEIGHT;
    bulletActive = 0;
    score = 0;
    gameOver = 0;
    resetObstacle();
}

/* ---------- INPUT ---------- */
/* 
   Controls:
   - Left  arrow : move left
   - Right arrow : move right
   - Up   arrow : move up (if possible)
   - Down arrow : move down (if possible)
   - Space      : shoot
*/

void handleInput() {
    if (_kbhit()) {
        int ch = _getch();

        /* Arrow keys come as two codes: 224 then actual code */
        if (ch == 224) {
            int code = _getch();
            if (code == 75 && playerLane > 0)          // LEFT
                playerLane--;
            else if (code == 77 && playerLane < LANES - 1) // RIGHT
                playerLane++;
            else if (code == 72 && playerRow > 1)      // UP
                playerRow--;
            else if (code == 80 && playerRow < TRACK_HEIGHT) // DOWN
                playerRow++;
        } else if (ch == ' ' || ch == 'z' || ch == 'Z') {
            // Shoot if no bullet active
            if (!bulletActive) {
                bulletActive = 1;
                bulletLane   = playerLane;
                bulletRow    = playerRow - 1; // start just above player
				//Beep(1200, 80);
				PlaySound(TEXT("shoot.wav"), NULL, SND_ASYNC | SND_FILENAME);
				//sleep(100);
            }
        }
    }
}

/* ---------- UPDATE ---------- */

void updateBullet() {
    if (!bulletActive)
        return;

    bulletRow--;

    // If bullet leaves top of screen
    if (bulletRow < 0) {
        bulletActive = 0;
        return;
    }

    // Bullet hits obstacle
    if (bulletLane == obstacleLane && bulletRow == obstacleRow) {
        bulletActive = 0;
        resetObstacle();
        score += 5; // reward more for shooting obstacle
        PlaySound(TEXT("impact.wav"), NULL, SND_ASYNC | SND_FILENAME);
    }
}

void updateObstacle() {
    obstacleRow++;

    // Obstacle goes past player without hitting
    if (obstacleRow > TRACK_HEIGHT) {
        resetObstacle();
        score += 1; // reward for dodging
    }
}

void checkCollision() {
    // Player and obstacle occupy same lane and row
    if (playerLane == obstacleLane && playerRow == obstacleRow) {
        gameOver = 1;
    }
}

/* ---------- DRAW ---------- */

void drawBorderTop() {
    printf("_");
    for (int i = 0; i < LANES * 4 + 1; i++) printf("_");
    printf("_\n");
}

void drawBorderBottom() {
    printf("_");
    for (int i = 0; i < LANES * 4 + 1; i++) printf("_");
    printf("_\n");
}

void drawGame() {
    clear_screen_fast();

    // Header with score and high score
    printf("  	GALAGA\n");
    printf("  Score: %d   High Score: %d\n", score, highScore);
    printf("  Move: ArrowKey | Shoot: Space\n\n");

    drawBorderTop();

    // Rows: 0 to TRACK_HEIGHT, where TRACK_HEIGHT is playerRow max
    for (int row = 0; row <= TRACK_HEIGHT; row++) {
        printf("|"); 

        for (int lane = 0; lane < LANES; lane++) {
            int cellDrawn = 0;

            // Player
            if (lane == playerLane && row == playerRow) {
                printf("  (O) ");
                cellDrawn = 1;
            }

            // Obstacle
            if (!cellDrawn && lane == obstacleLane && row == obstacleRow) {
                printf("  V ");
                cellDrawn = 1;
            }

            // Bullet
            if (!cellDrawn && bulletActive && lane == bulletLane && row == bulletRow) {
                printf("  ^ ");
                cellDrawn = 1;
            }

            if (!cellDrawn) {
                printf("    ");
            }
        }

        printf("|\n");
    }

    drawBorderBottom();
}

/* ---------- MAIN ---------- */

int main() {
    system("color 4F");
    srand((unsigned int)time(NULL));

    // Load existing high score
    loadHighScore();

    // Start background music (looped)
    PlaySound(TEXT("bg.wav"), NULL, SND_ASYNC | SND_LOOP | SND_FILENAME);

    resetGame();

    while (1) {
        handleInput();
        PlaySound(TEXT("bg.wav"), NULL, SND_ASYNC | SND_FILENAME | SND_NOSTOP);
        updateBullet();
        updateObstacle();
        checkCollision();
        drawGame();

        if (gameOver) {
            // Stop background sound and play impact
            //PlaySound(NULL, NULL, 0);
            PlaySound(TEXT("over.wav"), NULL, SND_ASYNC | SND_FILENAME);

            clear_screen_fast();
            printf("\n\n========== GAME OVER ==========\n");
            printf("\tYour score: %d\n", score);

            // High score logic
            if (score > highScore) {
                highScore = score;
                saveHighScore();
                printf("NEW HIGH SCORE!\n");
            } else {
                printf("High score: %d\n", highScore);
            }

            printf("\nRestart(Y/N)? ");
            int ch;
            do {
                ch = _getch();
            } while (ch != 'y' && ch != 'Y' && ch != 'n' && ch != 'N');

            if (ch == 'y' || ch == 'Y') {
                // Restart background music
                PlaySound(TEXT("bg.wav"), NULL, SND_ASYNC | SND_LOOP);
                resetGame();
            } else {
                break;
            }
        }

        sleep_ms(120);
    }

    // Stop all sounds
    PlaySound(NULL, NULL, 0);
    return 0;
}
