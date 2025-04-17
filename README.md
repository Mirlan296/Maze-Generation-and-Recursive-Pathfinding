Объяснение:



1. Константы и поля класса
java
Copy
private static final char WALL = '#';  // Символ стены
private static final char PATH = ' ';  // Символ прохода (пути)
private static final char SOLUTION = '.';  // Символ решения
private static final char START = 'S';  // Начальная точка
private static final char EXIT = 'E';  // Выходная точка

private char[][] maze;  // Двумерный массив для хранения лабиринта
private int width;  // Ширина лабиринта
private int height;  // Высота лабиринта
private int[] start = {1, 1};  // Координаты старта (x, y)
private int[] exit;  // Координаты выхода (x, y)
private Random random = new Random();  // Генератор случайных чисел
Здесь задаются символы, которыми будет отображаться лабиринт.

Поля width, height хранят размеры лабиринта.

start и exit — координаты входа и выхода.

random нужен для случайного перемешивания направлений при генерации.

2. Направления движения
java
Copy
private int[][] directions = {{0, -1}, {1, 0}, {0, 1}, {-1, 0}};
Это массив смещений для четырёх направлений:

{0, -1} — вверх

{1, 0} — вправо

{0, 1} — вниз

{-1, 0} — влево

3. Конструктор
java
Copy
public MazeSolver(int width, int height) {
    this.width = Math.max(width, 10);  // Минимальная ширина = 10
    this.height = Math.max(height, 10);  // Минимальная высота = 10
    this.exit = new int[]{this.width - 2, this.height - 2};  // Выход в правом нижнем углу
    this.maze = new char[this.height][this.width];  // Создаём массив лабиринта
    initializeMaze();  // Заполняем стенами
}
Устанавливает минимальный размер лабиринта 10x10.

Выход (exit) размещается в правом нижнем углу.

Вызывает initializeMaze(), чтобы заполнить лабиринт стенами.

4. Инициализация лабиринта (стены)
java
Copy
private void initializeMaze() {
    for (int y = 0; y < height; y++) {
        for (int x = 0; x < width; x++) {
            maze[y][x] = WALL;  // Каждая клетка — стена
        }
    }
    maze[start[1]][start[0]] = PATH;  // Стартовая точка — проход
    maze[exit[1]][exit[0]] = PATH;  // Выходная точка — проход
}
Весь лабиринт заполняется стенами (#).

Только старт и выход остаются пустыми ( ).

5. Генерация лабиринта (рекурсивный алгоритм)
java
Copy
public void generateMaze() {
    generateMazeRecursive(start[0], start[1]);  // Начинаем с точки старта
}
Запускает рекурсивную генерацию из стартовой точки.

6. Рекурсивная генерация (алгоритм "Backtracking")
java
Copy
private void generateMazeRecursive(int x, int y) {
    int[] dirs = {0, 1, 2, 3};  // 0=вверх, 1=вправо, 2=вниз, 3=влево
    shuffleArray(dirs);  // Перемешиваем направления
    
    for (int dir : dirs) {  // Проверяем каждое направление
        int nx = x + directions[dir][0] * 2;  // Координата через две клетки
        int ny = y + directions[dir][1] * 2;
        
        if (isInBounds(nx, ny) && maze[ny][nx] == WALL) {  // Если не вышли за границы и там стена
            maze[y + directions[dir][1]][x + directions[dir][0]] = PATH;  // Прорываем стену между текущей и следующей клеткой
            maze[ny][nx] = PATH;  // Делаем следующую клетку проходом
            generateMazeRecursive(nx, ny);  // Рекурсивно продолжаем
        }
    }
}
shuffleArray(dirs) — случайно перемешивает направления, чтобы лабиринт был разным.

nx, ny — координаты клетки через две (чтобы не было слишком узких проходов).

Если клетка (nx, ny) — стена (WALL), то:

Прорываем стену между текущей клеткой и следующей.

Рекурсивно вызываем generateMazeRecursive(nx, ny).

7. Решение лабиринта (рекурсивный поиск пути)
java
Copy
public boolean solveMaze() {
    boolean[][] visited = new boolean[height][width];  // Массив посещённых клеток
    return solveMazeRecursive(start[0], start[1], visited);
}
Создаётся массив visited, чтобы не ходить по кругу.

Запускается рекурсивный поиск пути.

Landysh, [17.04.2025 13:43]
8. Рекурсивный поиск пути (Depth-First Search)
java
Copy
private boolean solveMazeRecursive(int x, int y, boolean[][] visited) {
    if (!isInBounds(x, y)  maze[y][x] == WALL  visited[y][x]) {
        return false;  // Выход за границы, стена или уже были здесь → тупик
    }
    
    visited[y][x] = true;  // Помечаем клетку как посещённую
    
    if (x == exit[0] && y == exit[1]) {  // Если дошли до выхода
        maze[y][x] = EXIT;  // Помечаем выход
        return true;
    }
    
    for (int[] dir : directions) {  // Проверяем все 4 направления
        int nx = x + dir[0];
        int ny = y + dir[1];
        
        if (solveMazeRecursive(nx, ny, visited)) {  // Если путь найден
            if (!(x == start[0] && y == start[1])) {  // Если не стартовая точка
                maze[y][x] = SOLUTION;  // Помечаем путь точками
            }
            return true;
        }
    }
    
    return false;  // Если ни одно направление не привело к выходу
}
Проверка на тупик:

Если вышли за границы, наткнулись на стену или уже были в этой клетке → return false.

Помечаем клетку как посещённую (visited[y][x] = true).

Если дошли до выхода (x == exit[0] && y == exit[1]):

Помечаем клетку как 'E' и возвращаем true.

Рекурсивно проверяем все 4 направления:

Если в каком-то направлении путь найден (solveMazeRecursive(nx, ny, visited) == true), то:

Помечаем текущую клетку как часть решения ('.'), если это не старт.

Если ни одно направление не ведёт к выходу → return false.

9. Вспомогательные методы
java
Copy
private boolean isInBounds(int x, int y) {
    return x >= 0 && x < width && y >= 0 && y < height;
}

private void shuffleArray(int[] array) {
    for (int i = array.length - 1; i > 0; i--) {
        int index = random.nextInt(i + 1);
        int temp = array[index];
        array[index] = array[i];
        array[i] = temp;
    }
}
isInBounds — проверяет, что координаты (x, y) внутри лабиринта.

shuffleArray — перемешивает массив (алгоритм Фишера-Йетса).

10. Вывод лабиринта
java
Copy
public void printMaze() {
    for (int y = 0; y < height; y++) {
        for (int x = 0; x < width; x++) {
            if (x == start[0] && y == start[1]) {
                System.out.print(START);  // Стартовая точка — 'S'
            } else {
                System.out.print(maze[y][x]);  // Остальные символы как есть
            }
        }
        System.out.println();
    }
}
Выводит лабиринт в консоль, заменяя стартовую точку на 'S'.

11. Главный метод (main)
java
Copy
public static void main(String[] args) {
    Scanner scanner = new Scanner(System.in);
    
    System.out.print("Введите ширину лабиринта (минимум 10): ");
    int width = scanner.nextInt();
    
    System.out.print("Введите высоту лабиринта (минимум 10): ");
    int height = scanner.nextInt();
    
    MazeSolver mazeSolver = new MazeSolver(width, height);
    mazeSolver.generateMaze();
    
    System.out.println("\nСгенерированный лабиринт:");
    mazeSolver.printMaze();
    
    if (mazeSolver.solveMaze()) {
        System.out.println("\nЛабиринт с решением:");
        mazeSolver.printMaze();
    } else {
        System.out.println("\nРешение не найдено!");
    }
    
    scanner.close();
}
Запрашивает у пользователя ширину и высоту лабиринта.

Создаёт объект MazeSolver, генерирует лабиринт.

Выводит лабиринт.

Пытается найти решение:

Если решение найдено (solveMaze() == true), выводит лабиринт с путём ('.').

Если нет — сообщает об ошибке.

Логика рекурсивных алгоритмов
1. Генерация лабиринта (рекурсивный backtracking)
Выбираем случайное направление (перемешиваем dirs).

Проверяем клетку через две (nx = x + 2*dirX):

Если она внутри лабиринта и является стеной (WALL), то:

Прорываем стену между текущей и следующей клеткой.

Рекурсивно вызываем generateMazeRecursive(nx, ny).

Если все направления проверены → возвращаемся назад (backtracking).

Итог: Лабиринт генерируется так, что:

Все клетки соединены (нет изолированных областей).

Нет циклов (если не модифицировать алгоритм).

Landysh, [17.04.2025 13:43]
2. Поиск пути (Depth-First Search, DFS)
Помечаем текущую клетку как посещённую (visited[y][x] = true).

Если дошли до выхода → возвращаем true.

Проверяем все 4 направления:

Если путь найден (solveMazeRecursive(nx, ny, visited) == true), то:

Помечаем текущую клетку как часть решения ('.').

Возвращаем true.

Если ни одно направление не ведёт к выходу → возвращаем false.

Итог: Алгоритм находит путь от старта до выхода, помечая его точками ('.').

Вывод
Генерация лабиринта использует рекурсивный backtracking, прорывая стены в случайных направлениях.

Поиск пути использует рекурсивный DFS, помечая посещённые клетки и находя путь до выхода.

Оба алгоритма работают за O(N×M), где N и M — размеры лабиринта.
