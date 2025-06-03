#include <iostream>
#include <vector>
#include <limits>
#include <algorithm>
#include <ctime>
#include <cstdlib>

using namespace std;

const int BOARD_SIZE = 8;

enum Piece { EMPTY, PLAYER1, PLAYER2, KING1, KING2 };
enum Difficulty { EASY, MEDIUM, HARD };

class CheckersGame {
private:
    vector<vector<Piece>> board;
    Piece humanPlayer;
    Piece aiPlayer;
    Piece currentPlayer;
    Difficulty difficulty;
    bool gameAgainstAI;
    struct Move {
        int startX, startY;
        int endX, endY;
        int capturedX = -1, capturedY = -1;
        int score = 0;

        Move(int sx, int sy, int ex, int ey) : startX(sx), startY(sy), endX(ex), endY(ey) {}
    };

public:
    CheckersGame() : currentPlayer(PLAYER1), humanPlayer(PLAYER1), aiPlayer(PLAYER2),
                    difficulty(MEDIUM), gameAgainstAI(false) {
        board.resize(BOARD_SIZE, vector<Piece>(BOARD_SIZE, EMPTY));
        initializeBoard();
    }

    void initializeBoard() {
        for (int i = 0; i < BOARD_SIZE; i++) {
            for (int j = 0; j < BOARD_SIZE; j++) {
                if ((i + j) % 2 == 1) {
                    if (i < 3) board[i][j] = PLAYER2;
                    else if (i > 4) board[i][j] = PLAYER1;
                }
            }
        }
    }

    void displayBoard() {
        cout << "  ";
        for (char c = 'A'; c < 'A' + BOARD_SIZE; c++) {
            cout << c << " ";
        }
        cout << endl;

        for (int i = 0; i < BOARD_SIZE; i++) {
            cout << i + 1 << " ";
            for (int j = 0; j < BOARD_SIZE; j++) {
                char c;
                switch (board[i][j]) {
                    case PLAYER1: c = 'X'; break;
                    case PLAYER2: c = 'O'; break;
                    case KING1: c = 'K'; break;
                    case KING2: c = 'Q'; break;
                    default: c = ((i + j) % 2 == 1) ? '.' : ' '; break;
                }
                cout << c << " ";
            }
            cout << endl;
        }
    }

    bool isValidPosition(int x, int y) {
        return x >= 0 && x < BOARD_SIZE && y >= 0 && y < BOARD_SIZE;
    }

    bool isOwnPiece(int x, int y) {
        Piece piece = board[x][y];
        return (currentPlayer == PLAYER1 && (piece == PLAYER1 || piece == KING1)) ||
               (currentPlayer == PLAYER2 && (piece == PLAYER2 || piece == KING2));
    }

    bool isOpponentPiece(int x, int y) {
        if (!isValidPosition(x, y)) return false;
        Piece piece = board[x][y];
        return (currentPlayer == PLAYER1 && (piece == PLAYER2 || piece == KING2)) ||
               (currentPlayer == PLAYER2 && (piece == PLAYER1 || piece == KING1));
    }

    bool isEmpty(int x, int y) {
        return isValidPosition(x, y) && board[x][y] == EMPTY;
    }

    vector<Move> getPossibleMoves(int x, int y) {
        vector<Move> moves;
        if (!isValidPosition(x, y) || !isOwnPiece(x, y)) return moves;

        Piece piece = board[x][y];
        bool isKing = (piece == KING1 || piece == KING2);
        int forwardDir = (piece == PLAYER1 || piece == KING1) ? -1 : 1;

        if (isKing) {
            int directions[4][2] = {{1,1}, {1,-1}, {-1,1}, {-1,-1}};
            for (auto dir : directions) {
                int newX = x + dir[0];
                int newY = y + dir[1];
                if (isEmpty(newX, newY)) {
                    moves.emplace_back(x, y, newX, newY);
                }
            }
        } else {
            int newX = x + forwardDir;
            for (int newY : {y - 1, y + 1}) {
                if (isEmpty(newX, newY)) {
                    moves.emplace_back(x, y, newX, newY);
                }
            }
        }

        vector<Move> captureMoves = getCaptureMoves(x, y);
        moves.insert(moves.end(), captureMoves.begin(), captureMoves.end());

        return moves;
    }

    vector<Move> getCaptureMoves(int x, int y) {
        vector<Move> captureMoves;
        if (!isValidPosition(x, y) || !isOwnPiece(x, y)) return captureMoves;

        Piece piece = board[x][y];
        bool isKing = (piece == KING1 || piece == KING2);
        int forwardDir = (piece == PLAYER1 || piece == KING1) ? -1 : 1;
        int directions[4][2] = {{1,1}, {1,-1}, {-1,1}, {-1,-1}};
        for (auto dir : directions) {
            int jumpX = x + dir[0];
            int jumpY = y + dir[1];
            int landX = x + 2 * dir[0];
            int landY = y + 2 * dir[1];
            if (!isKing && ((piece == PLAYER1 && dir[0] > 0) || (piece == PLAYER2 && dir[0] < 0))) {
                continue;
            }

            if (isOpponentPiece(jumpX, jumpY) && isEmpty(landX, landY)) {
                Move move(x, y, landX, landY);
                move.capturedX = jumpX;
                move.capturedY = jumpY;
                captureMoves.push_back(move);
            }
        }

        return captureMoves;
    }

    vector<Move> getAllPossibleMovesForCurrentPlayer() {
        vector<Move> allMoves;
        bool hasCaptureMoves = false;
        for (int i = 0; i < BOARD_SIZE; i++) {
            for (int j = 0; j < BOARD_SIZE; j++) {
                if (isOwnPiece(i, j)) {
                    vector<Move> captureMoves = getCaptureMoves(i, j);
                    if (!captureMoves.empty()) {
                        hasCaptureMoves = true;
                        allMoves.insert(allMoves.end(), captureMoves.begin(), captureMoves.end());
                    }
                }
            }
        }

        if (hasCaptureMoves) {
            return allMoves;
        }
        for (int i = 0; i < BOARD_SIZE; i++) {
            for (int j = 0; j < BOARD_SIZE; j++) {
                if (isOwnPiece(i, j)) {
                    vector<Move> moves = getPossibleMoves(i, j);
                    allMoves.insert(allMoves.end(), moves.begin(), moves.end());
                }
            }
        }

        return allMoves;
    }

    bool makeMove(const Move& move) {
        if (!isValidPosition(move.startX, move.startY) || !isValidPosition(move.endX, move.endY)) {
            return false;
        }

        Piece piece = board[move.startX][move.startY];
        board[move.startX][move.startY] = EMPTY;
        board[move.endX][move.endY] = piece;
        if (move.capturedX != -1 && move.capturedY != -1) {
            board[move.capturedX][move.capturedY] = EMPTY;
        }
        if (move.endX == 0 && piece == PLAYER1) {
            board[move.endX][move.endY] = KING1;
        } else if (move.endX == BOARD_SIZE - 1 && piece == PLAYER2) {
            board[move.endX][move.endY] = KING2;
        }

        return true;
    }

    void undoMove(const Move& move) {
        Piece piece = board[move.endX][move.endY];
        board[move.endX][move.endY] = EMPTY;
        board[move.startX][move.startY] = piece;
        if (move.capturedX != -1 && move.capturedY != -1) {
            Piece opponent = (currentPlayer == PLAYER1) ? PLAYER2 : PLAYER1;
            if (piece == KING1 || piece == KING2) {
                opponent = (currentPlayer == PLAYER1) ? KING2 : KING1;
            }
            board[move.capturedX][move.capturedY] = opponent;
        }
    }

    int evaluateBoard() {
        int score = 0;
        int player1Pieces = 0;
        int player2Pieces = 0;
        int player1Kings = 0;
        int player2Kings = 0;

        for (int i = 0; i < BOARD_SIZE; i++) {
            for (int j = 0; j < BOARD_SIZE; j++) {
                switch (board[i][j]) {
                    case PLAYER1:
                        player1Pieces++;
                        score += (currentPlayer == PLAYER1) ? 5 : -5;
                        score += (currentPlayer == PLAYER1) ? (7 - i) : -(7 - i);
                        break;
                    case PLAYER2:
                        player2Pieces++;
                        score += (currentPlayer == PLAYER1) ? -5 : 5;
                        score += (currentPlayer == PLAYER1) ? -i : i;
                        break;
                    case KING1:
                        player1Kings++;
                        score += (currentPlayer == PLAYER1) ? 10 : -10;
                        break;
                    case KING2:
                        player2Kings++;
                        score += (currentPlayer == PLAYER1) ? -10 : 10;
                        break;
                    default:
                        break;
                }
            }
        }
        vector<Move> playerMoves = getAllPossibleMovesForCurrentPlayer();
        score += (currentPlayer == PLAYER1) ? playerMoves.size() : -playerMoves.size();

        score -= countIsolatedPieces(currentPlayer) * 2;
        score += countIsolatedPieces((currentPlayer == PLAYER1) ? PLAYER2 : PLAYER1) * 2;

        if (player1Pieces == 0 && player1Kings == 0) {
            return (currentPlayer == PLAYER2) ? 1000 : -1000;
        }
        if (player2Pieces == 0 && player2Kings == 0) {
            return (currentPlayer == PLAYER1) ? 1000 : -1000;
        }

        return score;
    }

    int countIsolatedPieces(Piece player) {
        int count = 0;
        for (int i = 0; i < BOARD_SIZE; i++) {
            for (int j = 0; j < BOARD_SIZE; j++) {
                if ((player == PLAYER1 && (board[i][j] == PLAYER1 || board[i][j] == KING1)) ||
                    (player == PLAYER2 && (board[i][j] == PLAYER2 || board[i][j] == KING2))) {

                    bool isolated = true;
                    for (int di = -1; di <= 1; di++) {
                        for (int dj = -1; dj <= 1; dj++) {
                            if (di == 0 && dj == 0) continue;
                            int ni = i + di;
                            int nj = j + dj;
                            if (isValidPosition(ni, nj)) {
                                if ((player == PLAYER1 && (board[ni][nj] == PLAYER1 || board[ni][nj] == KING1)) ||
                                    (player == PLAYER2 && (board[ni][nj] == PLAYER2 || board[ni][nj] == KING2))) {
                                    isolated = false;
                                    break;
                                }
                            }
                        }
                        if (!isolated) break;
                    }
                    if (isolated) count++;
                }
            }
        }
        return count;
    }

    Move findBestMove(int depth) {
        vector<Move> possibleMoves = getAllPossibleMovesForCurrentPlayer();
        if (possibleMoves.empty()) return Move(-1, -1, -1, -1);
        if (difficulty == EASY) {
            srand(time(0));
            return possibleMoves[rand() % possibleMoves.size()];
        }
        int searchDepth = (difficulty == MEDIUM) ? 3 : 5;

        Move bestMove(-1, -1, -1, -1);
        int bestScore = numeric_limits<int>::min();

        for (Move move : possibleMoves) {
            makeMove(move);
            currentPlayer = (currentPlayer == PLAYER1) ? PLAYER2 : PLAYER1;

            int score = minimax(searchDepth - 1, numeric_limits<int>::min(), numeric_limits<int>::max(), false);

            currentPlayer = (currentPlayer == PLAYER1) ? PLAYER2 : PLAYER1;
            undoMove(move);

            if (score > bestScore) {
                bestScore = score;
                bestMove = move;
            }
        }

        return bestMove;
    }

    int minimax(int depth, int alpha, int beta, bool maximizingPlayer) {
        if (depth == 0) {
            return evaluateBoard();
        }

        vector<Move> possibleMoves = getAllPossibleMovesForCurrentPlayer();
        if (possibleMoves.empty()) {
            return (maximizingPlayer) ? -1000 : 1000;
        }

        if (maximizingPlayer) {
            int maxEval = numeric_limits<int>::min();
            for (Move move : possibleMoves) {
                makeMove(move);
                currentPlayer = (currentPlayer == PLAYER1) ? PLAYER2 : PLAYER1;

                int eval = minimax(depth - 1, alpha, beta, false);

                currentPlayer = (currentPlayer == PLAYER1) ? PLAYER2 : PLAYER1;
                undoMove(move);

                maxEval = max(maxEval, eval);
                alpha = max(alpha, eval);
                if (beta <= alpha) {
                    break;
                }
            }
            return maxEval;
        } else {
            int minEval = numeric_limits<int>::max();
            for (Move move : possibleMoves) {
                makeMove(move);
                currentPlayer = (currentPlayer == PLAYER1) ? PLAYER2 : PLAYER1;

                int eval = minimax(depth - 1, alpha, beta, true);

                currentPlayer = (currentPlayer == PLAYER1) ? PLAYER2 : PLAYER1;
                undoMove(move);

                minEval = min(minEval, eval);
                beta = min(beta, eval);
                if (beta <= alpha) {
                    break;
                }
            }
            return minEval;
        }
    }

    bool isGameOver() {
        bool player1HasPieces = false;
        bool player2HasPieces = false;

        for (int i = 0; i < BOARD_SIZE; i++) {
            for (int j = 0; j < BOARD_SIZE; j++) {
                if (board[i][j] == PLAYER1 || board[i][j] == KING1) {
                    player1HasPieces = true;
                } else if (board[i][j] == PLAYER2 || board[i][j] == KING2) {
                    player2HasPieces = true;
                }
            }
        }

        if (!player1HasPieces || !player2HasPieces) {
            return true;
        }
        vector<Move> moves = getAllPossibleMovesForCurrentPlayer();
        return moves.empty();
    }

    void play() {
        displayMainMenu();

        while (true) {
            displayBoard();

            if (isGameOver()) {
                cout << "Игра окончена! ";
                if (currentPlayer == humanPlayer) {
                    cout << "Вы проиграли." << endl;
                } else {
                    cout << "Вы победили!" << endl;
                }
                break;
            }

            if (currentPlayer == humanPlayer) {
                humanMove();
            } else if (gameAgainstAI) {
                cout << "Ход ИИ" << endl;
                Move aiMove = findBestMove(5);
                if (aiMove.startX != -1) {
                    cout << "ИИ делает ход: "
                         << char(aiMove.startY + 'A') << aiMove.startX + 1 << " -> "
                         << char(aiMove.endY + 'A') << aiMove.endX + 1 << endl;
                    makeMove(aiMove);
                }
                switchPlayer();
            } else {
                humanMove();
            }
        }
    }

    void humanMove() {
        cout << "Игрок " << ((currentPlayer == PLAYER1) ? "1 (X)" : "2 (O)") << ", ваш ход." << endl;
        cout << "Введите ход (например, A3 B4) или 'menu' для вызова меню: ";

        string input;
        cin >> input;

        if (input == "menu") {
            displayInGameMenu();
            return;
        }

        char startCol = toupper(input[0]);
        int startRow = input[1] - '0';

        cin >> input;
        char endCol = toupper(input[0]);
        int endRow = input[1] - '0';

        int startX = startRow - 1;
        int startY = startCol - 'A';
        int endX = endRow - 1;
        int endY = endCol - 'A';

        vector<Move> allMoves = getAllPossibleMovesForCurrentPlayer();
        bool isValid = false;
        Move selectedMove(startX, startY, endX, endY);

        for (const Move& move : allMoves) {
            if (move.startX == startX && move.startY == startY &&
                move.endX == endX && move.endY == endY) {
                isValid = true;
                selectedMove = move;
                break;
            }
        }

        if (isValid) {
            makeMove(selectedMove);
            if (selectedMove.capturedX != -1) {
                vector<Move> furtherCaptures = getCaptureMoves(selectedMove.endX, selectedMove.endY);
                if (!furtherCaptures.empty()) {
                    displayBoard();
                    cout << "Вы можете продолжить захват с позиции "
                         << char(selectedMove.endY + 'A') << selectedMove.endX + 1 << endl;
                    currentPlayer = (currentPlayer == PLAYER1) ? PLAYER1 : PLAYER2;
                    return;
                }
            }
            switchPlayer();
        } else {
            cout << "Неверный ход. Попробуйте снова." << endl;
            humanMove();
        }
    }

    void displayMainMenu() {
        cout << "=== ШАШКИ ===" << endl;
        cout << "1. Играть против компьютера" << endl;
        cout << "2. Играть вдвоем" << endl;
        cout << "3. Выход" << endl;
        cout << "Выберите вариант: ";

        int choice;
        cin >> choice;

        switch (choice) {
            case 1:
                gameAgainstAI = true;
                selectSide();
                selectDifficulty();
                break;
            case 2:
                gameAgainstAI = false;
                humanPlayer = PLAYER1;
                aiPlayer = PLAYER2;
                break;
            case 3:
                exit(0);
            default:
                cout << "Неверный выбор. Попробуйте снова." << endl;
                displayMainMenu();
        }
    }

    void displayInGameMenu() {
        cout << "=== МЕНЮ ===" << endl;
        cout << "1. Продолжить игру" << endl;
        cout << "2. Изменить сложность" << endl;
        cout << "3. Главное меню" << endl;
        cout << "4. Выход" << endl;
        cout << "Выберите вариант: ";

        int choice;
        cin >> choice;

        switch (choice) {
            case 1:
                return;
            case 2:
                selectDifficulty();
                break;
            case 3:
                initializeBoard();
                currentPlayer = PLAYER1;
                displayMainMenu();
                break;
            case 4:
                exit(0);
            default:
                cout << "Неверный выбор. Попробуйте снова." << endl;
                displayInGameMenu();
        }
    }
    void selectSide() {
        cout << "Выберите сторону:" << endl;
        cout << "1. X (играют первыми)" << endl;
        cout << "2. O (играют вторыми)" << endl;
        cout << "Выберите вариант: ";

        int choice;
        cin >> choice;

        if (choice == 1) {
            humanPlayer = PLAYER1;
            aiPlayer = PLAYER2;
            currentPlayer = PLAYER1;
        } else if (choice == 2) {
            humanPlayer = PLAYER2;
            aiPlayer = PLAYER1;
            currentPlayer = PLAYER1;
        } else {
            cout << "Неверный выбор. Попробуйте снова." << endl;
            selectSide();
        }
    }

    void selectDifficulty() {
        cout << "Выберите сложность ИИ:" << endl;
        cout << "1. Новичок" << endl;
        cout << "2. Средний" << endl;
        cout << "3. Эксперт" << endl;
        cout << "Выберите вариант: ";

        int choice;
        cin >> choice;

        if (choice >= 1 && choice <= 3) {
            difficulty = static_cast<Difficulty>(choice - 1);
        } else {
            cout << "Неверный выбор. Попробуйте снова." << endl;
            selectDifficulty();
        }
    }

    void switchPlayer() {
        currentPlayer = (currentPlayer == PLAYER1) ? PLAYER2 : PLAYER1;
    }
};

int main() {
    CheckersGame game;
    game.play();
    return 0;
}
