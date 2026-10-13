#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <ctype.h>

// Прототипы функций
void clear_screen();
void print_board(char **board, int size);
int check_win(char **board, int size, char symbol);
int is_win_possible(char **board, int size);
void player_move(char **board, int size, char symbol);
void bot_move(char **board, int size, char bot_symbol, char player_symbol);

int main() {
    int size;
    srand(time(NULL));

    printf("Введите размер поля (x): ");
    while (scanf("%d", &size) != 1 || size < 3) {
        while(getchar() != '\n');
        printf("Некорректный размер. Введите целое число >= 3: ");
    }

    char **board = (char**)malloc(size * sizeof(char*));
    for (int i = 0; i < size; i++) {
        board[i] = (char*)malloc(size * sizeof(char));
        for (int j = 0; j < size; j++) {
            board[i][j] = ' ';
        }
    }

    int mode;
    printf("Выберите режим (1 - на двоих, 2 - с ботом): ");
    while (scanf("%d", &mode) != 1 || (mode != 1 && mode != 2)) {
        while(getchar() != '\n');
        printf("Некорректный ввод. Введите 1 или 2: ");
    }

    char user_sym = ' ', bot_sym = ' ';

    if (mode == 2) {
        printf("За кого вы хотите играть? (X или O): ");
        while (scanf(" %c", &user_sym) != 1 || (toupper(user_sym) != 'X' && toupper(user_sym) != 'O')) {
            while(getchar() != '\n');
            printf("Некорректный ввод. Выберите X или O: ");
        }
        user_sym = toupper(user_sym);
        bot_sym = (user_sym == 'X') ? 'O' : 'X';
        printf("Вы играете за %c, бот играет за %c\n", user_sym, bot_sym);
        printf("Нажмите Enter для начала игры...");
        while (getchar() != '\n'); // пауза
        getchar();
    } else {
        printf("Нажмите Enter для начала игры...");
        while (getchar() != '\n');
        getchar();
    }

    // 4. Игровой цикл
    int turn = 0;
    char current_sym = 'X';

    while (1) {
        // Очищаем экран и показываем актуальное поле
        clear_screen();
        print_board(board, size);

        // Ход текущего игрока
        if (mode == 2 && current_sym == bot_sym) {
            printf("Бот ходит...\n");
            bot_move(board, size, bot_sym, user_sym);
        } else {
            char move_sym = (mode == 2) ? user_sym : current_sym;
            player_move(board, size, move_sym);
        }

        // Определяем, какой символ только что ходил
        char check_sym = (mode == 2 && current_sym == bot_sym) ? bot_sym
                        : ((mode == 2) ? user_sym : current_sym);

        // Проверка победы
        if (check_win(board, size, check_sym)) {
            clear_screen();
            print_board(board, size);
            if (mode == 2) {
                if (check_sym == user_sym) printf("Поздравляем! Вы победили!\n");
                else printf("Бот победил! Попробуйте еще раз.\n");
            } else {
                printf("Игрок %c победил!\n", check_sym);
            }
            break;
        }

        // Проверка: возможна ли ещё победа хотя бы для одного игрока?
        if (!is_win_possible(board, size)) {
            clear_screen();
            print_board(board, size);
            printf("Победа больше невозможна. Игра завершается вничью!\n");
            break;
        }

        // Смена хода
        turn = 1 - turn;
        current_sym = (turn == 0) ? 'X' : 'O';
    }

    // 5. Освобождение памяти
    for (int i = 0; i < size; i++) {
        free(board[i]);
    }
    free(board);

    return 0;
}

// Очистка экрана (кроссплатформенно)
void clear_screen() {
#ifdef _WIN32
    system("cls");
#else
    system("clear");
#endif
}

// Функция вывода поля на экран
void print_board(char **board, int size) {
    printf("\n   ");
    for (int j = 0; j < size; j++) {
        printf("%-2d ", j + 1);
    }
    printf("\n");
    for (int i = 0; i < size; i++) {
        printf("%-2d ", i + 1);
        for (int j = 0; j < size; j++) {
            printf(" %c ", board[i][j]);
        }
        printf("\n");
    }
    printf("\n");
}

// Функция проверки победы
int check_win(char **board, int size, char symbol) {
    // Проверка строк
    for (int i = 0; i < size; i++) {
        int count = 0;
        for (int j = 0; j < size; j++) {
            if (board[i][j] == symbol) count++;
        }
        if (count == size) return 1;
    }

    // Проверка столбцов
    for (int j = 0; j < size; j++) {
        int count = 0;
        for (int i = 0; i < size; i++) {
            if (board[i][j] == symbol) count++;
        }
        if (count == size) return 1;
    }

    // Главная диагональ
    int count = 0;
    for (int i = 0; i < size; i++) {
        if (board[i][i] == symbol) count++;
    }
    if (count == size) return 1;

    // Побочная диагональ
    count = 0;
    for (int i = 0; i < size; i++) {
        if (board[i][size - 1 - i] == symbol) count++;
    }
    if (count == size) return 1;

    return 0;
}

// Проверка: возможна ли победа хотя бы теоретически?
// Возвращает 1, если есть хотя бы одна линия, которую кто-то может заполнить.
// Возвращает 0, если все линии заблокированы (содержат и X, и O).
int is_win_possible(char **board, int size) {
    // Проверка строк
    for (int i = 0; i < size; i++) {
        int has_x = 0, has_o = 0;
        for (int j = 0; j < size; j++) {
            if (board[i][j] == 'X') has_x = 1;
            if (board[i][j] == 'O') has_o = 1;
        }
        if (!(has_x && has_o)) return 1;
    }

    // Проверка столбцов
    for (int j = 0; j < size; j++) {
        int has_x = 0, has_o = 0;
        for (int i = 0; i < size; i++) {
            if (board[i][j] == 'X') has_x = 1;
            if (board[i][j] == 'O') has_o = 1;
        }
        if (!(has_x && has_o)) return 1;
    }

    // Главная диагональ
    int has_x = 0, has_o = 0;
    for (int i = 0; i < size; i++) {
        if (board[i][i] == 'X') has_x = 1;
        if (board[i][i] == 'O') has_o = 1;
    }
    if (!(has_x && has_o)) return 1;

    // Побочная диагональ
    has_x = 0; has_o = 0;
    for (int i = 0; i < size; i++) {
        if (board[i][size - 1 - i] == 'X') has_x = 1;
        if (board[i][size - 1 - i] == 'O') has_o = 1;
    }
    if (!(has_x && has_o)) return 1;

    return 0; // все линии заблокированы
}

// Ход игрока
void player_move(char **board, int size, char symbol) {
    int row, col;
    while (1) {
        printf("Ход игрока %c (строка и столбец от 1 до %d): ", symbol, size);
        if (scanf("%d %d", &row, &col) != 2) {
            while(getchar() != '\n');
            printf("Некорректный ввод. Попробуйте снова.\n");
            continue;
        }
        row--; col--;
        if (row >= 0 && row < size && col >= 0 && col < size && board[row][col] == ' ') {
            board[row][col] = symbol;
            break;
        } else {
            printf("Некорректный ход (клетка занята или вне поля). Попробуйте снова.\n");
        }
    }
}

// Ход бота
void bot_move(char **board, int size, char bot_symbol, char player_symbol) {
    // 1. Попытка выиграть
    for (int i = 0; i < size; i++) {
        for (int j = 0; j < size; j++) {
            if (board[i][j] == ' ') {
                board[i][j] = bot_symbol;
                if (check_win(board, size, bot_symbol)) return;
                board[i][j] = ' ';
            }
        }
    }

    // 2. Блокировка игрока
    for (int i = 0; i < size; i++) {
        for (int j = 0; j < size; j++) {
            if (board[i][j] == ' ') {
                board[i][j] = player_symbol;
                if (check_win(board, size, player_symbol)) {
                    board[i][j] = bot_symbol;
                    return;
                }
                board[i][j] = ' ';
            }
        }
    }

    // 3. Случайный ход
    int empty_cells = 0;
    for (int i = 0; i < size; i++)
        for (int j = 0; j < size; j++)
            if (board[i][j] == ' ') empty_cells++;

    if (empty_cells == 0) return;

    int move = rand() % empty_cells;
    for (int i = 0; i < size; i++) {
        for (int j = 0; j < size; j++) {
            if (board[i][j] == ' ') {
                if (move == 0) {
                    board[i][j] = bot_symbol;
                    return;
                }
                move--;
            }
        }
    }
}
