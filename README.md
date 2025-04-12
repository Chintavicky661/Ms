2.a.Minimum-cost spanning

#include <stdio.h>
#include <stdlib.h>
#define INF 9999
#define MAX 20
int G[MAX][MAX], S[MAX][MAX], n;
int prims() {
    int cost[MAX][MAX], dist[MAX], from[MAX], visited[MAX] = {1};
    int min, u, v, total = 0, edges = n - 1;
    for (int i = 0; i < n; i++) {
        dist[i] = INF;
        for (int j = 0; j < n; j++) {
            cost[i][j] = (G[i][j] == 0) ? INF : G[i][j];
            S[i][j] = 0;
        }
    }
    for (int i = 1; i < n; i++) {
        dist[i] = cost[0][i];
        from[i] = 0;
    }
    while (edges--) {
        min = INF;
        for (int i = 1; i < n; i++)
            if (!visited[i] && dist[i] < min)
                min = dist[i], v = i;
        u = from[v];
        S[u][v] = S[v][u] = dist[v];
        visited[v] = 1;
        total += cost[u][v];
        for (int i = 1; i < n; i++)
            if (!visited[i] && cost[v][i] < dist[i])
                dist[i] = cost[v][i], from[i] = v;
    }
    return total;
}
int main() {
    printf("Enter no. of vertices: ");
    scanf("%d", &n);
    printf("\nEnter the adjacency matrix:\n");
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            scanf("%d", &G[i][j]);
    int total_cost = prims();
    printf("\nSpanning tree matrix:\n");
    for (int i = 0; i < n; i++) {
        printf("\n");
        for (int j = 0; j < n; j++)
            printf("%d\t", S[i][j]);
    }
    printf("\n\nTotal cost of the spanning tree = %d\n", total_cost);
}
....
4.a or 6.b.Hamiltonian Cycle problem with Backtracking

#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#define V 3
int isSafe(int v, int graph[V][V], int path[], int pos) {
    if (!graph[path[pos - 1]][v]) return 0;
    for (int i = 0; i < pos; i++)
        if (path[i] == v) return 0;
    return 1;
}
int solve(int graph[V][V], int path[], int pos) {
    if (pos == V)
        return graph[path[pos - 1]][path[0]];
    for (int v = 1; v < V; v++) {
        if (isSafe(v, graph, path, pos)) {
            path[pos] = v;
            if (solve(graph, path, pos + 1)) return 1;
            path[pos] = -1;
        }
    }
    return 0;
}
int hamiltonianCycle(int graph[V][V]) {
    int path[V];
    for (int i = 0; i < V; i++) path[i] = -1;
    path[0] = 0;
    if (!solve(graph, path, 1)) return 0;
    printf("Hamiltonian Cycle exists: ");
    for (int i = 0; i < V; i++) printf("%d ", path[i]);
    printf("\n");
    return 1;
}
int main() {
    int graph[V][V];
    clock_t start, end;
    double time;
    printf("Enter the adjacency matrix for the graph (size %d x %d):\n", V, V);
    for (int i = 0; i < V; i++)
        for (int j = 0; j < V; j++)
            scanf("%d", &graph[i][j]);
    start = clock();
    if (!hamiltonianCycle(graph))
        printf("No Hamiltonian Cycle exists\n");
    end = clock();
    time = ((double)(end - start)) / CLOCKS_PER_SEC;
    printf("Time taken to find Hamiltonian Cycle: %f seconds\n", time);
}
###################################################################################################################################
8.a.8-Queens problem with Backtracking

#include <stdio.h>
#include <time.h>
#define N 8
int board[N][N];
int isSafe(int row, int col) {
    for (int i = 0; i < row; i++)
        if (board[i][col] || (col - (row - i) >= 0 && board[i][col - (row - i)]) || (col + (row - i) < N && board[i][col + (row - i)]))
            return 0;
    return 1;
}
int solve(int row) {
    if (row == N) return 1;
    for (int col = 0; col < N; col++) {
        if (isSafe(row, col)) {
            board[row][col] = 1;
            if (solve(row + 1)) return 1;
            board[row][col] = 0;
        }
    }
    return 0;
}
int main() {
    if (solve(0)) {
        printf("Solution:\n");
        for (int i = 0; i < N; i++, printf("\n"))
            for (int j = 0; j < N; j++)
                printf("%d ", board[i][j]);
    } else {
        printf("No solution exists.\n");
    }
}



....
#include <stdio.h>
#include <math.h>

#define SIZE 8

int queen[SIZE];
int solutionCount = 0;

int isSafe(int row, int col) {
    for (int i = 0; i < row; i++) {
        if (queen[i] == col || abs(queen[i] - col) == abs(i - row))
            return 0;
    }
    return 1;
}

void printBoard() {
    solutionCount++;
    printf("Solution %d:\n", solutionCount);
    for (int i = 0; i < SIZE; i++) {
        for (int j = 0; j < SIZE; j++) {
            if (queen[i] == j)
                printf("Q ");
            else
                printf(". ");
        }
        printf("\n");
    }
    printf("\n");
}

void solve(int row) {
    for (int col = 0; col < SIZE; col++) {
        if (isSafe(row, col)) {
            queen[row] = col;
            if (row == SIZE - 1)
                printBoard();
            else
                solve(row + 1);
        }
    }
}

int main() {
    solve(0);
    printf("Total solutions: %d\n", solutionCount);
    return 0;
}
