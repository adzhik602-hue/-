#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <ctype.h>



void print_board(char **board, int size);
int check_win(char **board, int size, char symbol);
int is_full(char **board, int size);
void player_move(char **board, int size, char symbol);
void bot_move(char **board, int size, char bot_symbol, char player_symbol);




int main() {
    int size;
    srand(time(NULL));

    
    printf("Введи размер поля (x): ");
    while (scanf("%d", &size) != 1 || size < 3) {
        while(getchar() != '\n');
        printf("Не то Введи целое число >= 3: ");
    }




    char **board = (char**)malloc(size * sizeof(char*));
    for (int i = 0; i < size; i++) {
        board[i] = (char*)malloc(size * sizeof(char));
        for (int j = 0; j < size; j++) {
            board[i][j] = ' ';
        }
    }





    int mode;
    printf("Выбери режим 1 - с человеком ( или не очень ), 2 - со мной : ");
    while (scanf("%d", &mode) != 1 || (mode != 1 && mode != 2)) {
        while(getchar() != '\n');
        printf("... 1 или 2. ни чего другого от тебя не жду: ");
    }

    char user_sym = ' ', bot_sym = ' ';







    if (mode == 2) {
        printf("Кто ты есть, воин?: ");
        while (scanf(" %c", &user_sym) != 1 || (toupper(user_sym) != 'X' && toupper(user_sym) != 'O')) {
            while(getchar() != '\n');
            printf("Ты дурак?): ");
        }
        user_sym = toupper(user_sym);
        bot_sym = (user_sym == 'X') ? 'O' : 'X';
        printf("ты  %c, я %c\n", user_sym, bot_sym);
    }





    int turn = 0; // 0 - ты (X), 1 - твой друг (O)
    char current_sym = 'X';

    while (1) {
        print_board(board, size);






        if (mode == 2 && current_sym == bot_sym) {
            printf("Подожди. я думаю\n");
            bot_move(board, size, bot_sym, user_sym);
        } else {
            char move_sym = (mode == 2) ? user_sym : current_sym;
            player_move(board, size, move_sym);
        }








        char check_sym = (mode == 2 && current_sym == bot_sym) ? bot_sym : ((mode == 2) ? user_sym : current_sym);
        if (check_win(board, size, check_sym)) {
            print_board(board, size);
            if (mode == 2) {
                if (check_sym == user_sym) printf("Поздравляю, ты сильнее меня)\n");
                else printf("я победил).\n");
            } else {
                printf("ты %c победил\n", check_sym);
            }
            break;
        }






        
        if (is_full(board, size)) {
            print_board(board, size);
            printf("Будь сильнее, а пока ты на ровне со мной \n");
            break;
        }





        turn = 1 - turn;
        current_sym = (turn == 0) ? 'X' : 'O';
    }





    for (int i = 0; i < size; i++) {
        free(board[i]);
    }
    free(board);

    return 0;
}







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









int check_win(char **board, int size, char symbol) {
    for (int i = 0; i < size; i++) {
        int count = 0;
        for (int j = 0; j < size; j++) {
            if (board[i][j] == symbol) count++;
        }
        if (count == size) return 1;
    }







    for (int j = 0; j < size; j++) {
        int count = 0;
        for (int i = 0; i < size; i++) {
            if (board[i][j] == symbol) count++;
        }
        if (count == size) return 1;
    }









    int count = 0;
    for (int i = 0; i < size; i++) {
        if (board[i][i] == symbol) count++;
    }
    if (count == size) return 1;
    count = 0;
    for (int i = 0; i < size; i++) {
        if (board[i][size - 1 - i] == symbol) count++;
    }
    if (count == size) return 1;

    return 0;
}







int is_full(char **board, int size) {
    for (int i = 0; i < size; i++) {
        for (int j = 0; j < size; j++) {
            if (board[i][j] == ' ') return 0;
        }
    }
    return 1;
}






void player_move(char **board, int size, char symbol) {
    int row, col;
    while (1) {
        printf("Ход игрока %c (строка и столбец от 1 до %d): ", symbol, size);
        if (scanf("%d %d", &row, &col) != 2) {
            while(getchar() != '\n'); // 
            printf("Подумай ещё\n");
            continue;
        }
        row--; col--; 
        if (row >= 0 && row < size && col >= 0 && col < size && board[row][col] == ' ') {
            board[row][col] = symbol;
            break;
        } else {
            printf("Ты явно делаешь что то не так\n");
        }
    }
}









void bot_move(char **board, int size, char bot_symbol, char player_symbol) {



    for (int i = 0; i < size; i++) {
        for (int j = 0; j < size; j++) {
            if (board[i][j] == ' ') {
                board[i][j] = bot_symbol;
                if (check_win(board, size, bot_symbol)) return;
                board[i][j] = ' ';
            }
        }
    }









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
