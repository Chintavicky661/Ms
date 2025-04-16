Lexical analysis using lex tool:

%{
#include<stdio.h>
%}

%%

[ \t]+                  ;
[0-9]*\.[0-9]+|[0-9]+  { printf("\n%s is Number", yytext); }
#.*                     { printf("\n%s is Comment", yytext); }
[a-zA-Z][a-zA-Z0-9]*    { printf("\n%s is Identifier", yytext); }
\"[^\"\n]*\"            { printf("\n%s is String", yytext); }
[*%+\-]                 { printf("\nOperator: %s", yytext); }
[(){};]                 { printf("\nSpecial Character: %s", yytext); }
\n                      { ECHO; }

%%

int main() {
    yylex(); 
}

int yywrap() {
    return 1;
}


ignore redundant spaces:

%{
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
void print_token(const char *type, const char *value);
%}

%%

"if"|"else"|"while"|"for"|"return"|"function" { print_token("KEYWORD", yytext); }
[a-zA-Z_][a-zA-Z0-9_]* { print_token("IDENTIFIER", yytext); }
[0-9]+ { print_token("NUMBER", yytext); }
[+\-*/] { print_token("OPERATOR", yytext); }
[<>] { print_token("COMPARE OPERATOR", yytext); }
"=" { print_token("ASSIGNMENT", yytext); }
[()] { print_token("PARENTHESIS", yytext); }
[{}] { print_token("BRACE", yytext); }
[;] { print_token("SEMICOLON", yytext); }
[ \t\n\r]+ { /* Ignore whitespace */ }
. { printf("Unknown character: %s\n", yytext); }

%%

void print_token(const char *type, const char *value) {
    printf("Token Type: %-25s Value: %s\n", type, value);
}

int main(int argc, char **argv) {
    if (argc > 1) {
        FILE *file = fopen(argv[1], "r");
        if (!file) {
            perror("fopen");
            return 1;
        }
        yyin = file;
    }
    yylex();
    return 0;
}

int yywrap(void) {
    return 1;
}





FIRST AND FLOW;

#include <stdio.h>
#include <string.h>
#include <ctype.h>

int n, m = 0;
char a[10][10], f[20];

void follow(char c);
void first(char c);

int main() {
    int i, z;
    char c;

    printf("Enter the number of productions: ");
    scanf("%d", &n);

    printf("Enter the productions (use $ for epsilon):\n");
    for(i = 0; i < n; i++) {
        scanf("%s", a[i]);
    }

    do {
        m = 0;
        printf("\nEnter the element whose FIRST & FOLLOW is to be found: ");
        scanf(" %c", &c);

        first(c);
        printf("FIRST(%c) = { ", c);
        for(i = 0; i < m; i++) {
            printf("%c ", f[i]);
        }
        printf("}\n");

        int old_m = m;
        follow(c);
        printf("FOLLOW(%c) = { ", c);
        for(i = old_m; i < m; i++) {
            printf("%c ", f[i]);
        }
        printf("}\n");

        printf("Do you want to continue? (1 for Yes / 0 for No): ");
        scanf("%d", &z);
    } while(z == 1);

    return 0;
}

void follow(char c) {
    int i, j;

    if(a[0][0] == c) {
        f[m++] = '$'; // Add end symbol to FOLLOW(start symbol)
    }

    for(i = 0; i < n; i++) {
        for(j = 2; j < strlen(a[i]); j++) {
            if(a[i][j] == c) {
                if(a[i][j+1] != '\0') {
                    first(a[i][j+1]);
                } else if(c != a[i][0]) {
                    follow(a[i][0]);
                }
            }
        }
    }
}

void first(char c) {
    int k;

    if(!isupper(c)) {
        f[m++] = c;
        return;
    }

    for(k = 0; k < n; k++) {
        if(a[k][0] == c) {
            if(a[k][2] == '$') {
                f[m++] = '$';
            } else {
                first(a[k][2]);
            }
        }
    }
}



precedence parser:

#include <stdio.h>
#include <string.h>

char stack[20];
int top = -1;

void push(char item) {
    if (top >= 19) {
        printf("STACK OVERFLOW\n");
        return;
    }
    stack[++top] = item;
    printf("Push: %c\n", item);
}

char pop() {
    if (top <= -1) {
        printf("STACK UNDERFLOW\n");
        return '\0';
    }
    char c = stack[top--];
    printf("Popped element: %c\n", c);
    return c;
}

char TOS() {
    if (top <= -1) {
        printf("STACK EMPTY\n");
        return '\0';
    }
    return stack[top];
}

int convert(char item) {
    switch (item) {
        case 'i': return 0;
        case '+': return 1;
        case '*': return 2;
        case '$': return 3;
        default: return -1;
    }
}

int main() {
    char pt[4][4] = {
        { '-', '>', '>', '>' },
        { '<', '>', '<', '>' },
        { '<', '>', '>', '>' },
        { '<', '<', '<', '1' }
    };

    char input[20];
    int idx = 0;

    printf("Enter input with $ at the end (e.g., i+i*i$): ");
    scanf("%s", input);

    push('$');

    while (idx < strlen(input)) {
        char topSym = TOS();
        char current = input[idx];

        int row = convert(topSym);
        int col = convert(current);

        if (row == -1 || col == -1) {
            printf("Invalid symbol encountered: %c\n", current);
            return 1;
        }

        if (topSym == '$' && current == '$') {
            printf("SUCCESS\n");
            return 0;
        }

        char action = pt[row][col];

        if (action == '<' || action == '-') {
            push(current);
            idx++;
        } else if (action == '>') {
            pop();
        } else if (action == '1') {
            printf("SUCCESS\n");
            return 0;
        } else {
            printf("FAILURE (Invalid action)\n");
            return 1;
        }
    }

    printf("FAILURE\n");
    return 1;
}

RECURSIVE DESCENT PARS:

#include <stdio.h>
#include <ctype.h>
#include <string.h>

void Tp();
void Ep();
void E();
void T();
void check();

int count, flag;
char expr[50];

int main() {
    count = 0;
    flag = 0;

    printf("\nENTER AN ALGEBRAIC EXPRESSION:\t");
    scanf("%s", expr);

    E();

    if ((strlen(expr) == count) && (flag == 0))
        printf("\nTHE EXPRESSION %s IS VALID\n", expr);
    else
        printf("\nTHE EXPRESSION %s IS INVALID\n", expr);

    return 0;
}

void E() {
    T();
    Ep();
}

void Ep() {
    if (expr[count] == '+') {
        count++;
        T();
        Ep();
    }
}

void T() {
    check();
    Tp();
}

void Tp() {
    if (expr[count] == '*') {
        count++;
        check();
        Tp();
    }
}

void check() {
    if (isalnum(expr[count])) {
        count++;
    } else if (expr[count] == '(') {
        count++;
        E();
        if (expr[count] == ')') {
            count++;
        } else {
            flag = 1;
        }
    } else {
        flag = 1;
    }
}


LL(1) parse:

#include <stdio.h>
#include <string.h>

int stack[20], top = -1;

void push(int item) {
    if (top >= 19) {
        printf("Stack Overflow\n");
        return;
    }
    stack[++top] = item;
}

int pop() {
    if (top <= -1) {
        printf("Stack Underflow\n");
        return -1;
    }
    return stack[top--];
}

char convert(int item) {
    switch (item) {
        case 0: return 'E';
        case 1: return 'e';
        case 2: return 'T';
        case 3: return 't';
        case 4: return 'F';
        case 5: return 'i';
        case 6: return '+';
        case 7: return '*';
        case 8: return '(';
        case 9: return ')';
        case 10: return '$';
        default: return '?';
    }
}

int main() {
    int m[5][11] = {0};

    m[0][5] = m[0][8] = 21;
    m[1][6] = 621;
    m[1][9] = m[1][10] = -2;
    m[2][5] = m[2][8] = 43;
    m[3][6] = m[3][9] = m[3][10] = -2;
    m[3][7] = 743;
    m[4][5] = 5;
    m[4][8] = 809;

    char ips[20];
    int ip[20], i, j, k, a, b, t;

    printf("\nEnter the input string with $ at the end (e.g., i+i*i$):\n");
    scanf("%s", ips);

    for (i = 0; i < strlen(ips); i++) {
        switch (ips[i]) {
            case 'E': k = 0; break;
            case 'e': k = 1; break;
            case 'T': k = 2; break;
            case 't': k = 3; break;
            case 'F': k = 4; break;
            case 'i': k = 5; break;
            case '+': k = 6; break;
            case '*': k = 7; break;
            case '(': k = 8; break;
            case ')': k = 9; break;
            case '$': k = 10; break;
            default: 
                printf("Invalid input\n");
                return 1;
        }
        ip[i] = k;
    }
    ip[i] = -1;

    push(10);
    push(0);

    i = 0;
    printf("\n\tSTACK\t\tINPUT\n");

    while (1) {
        printf("\t");
        for (j = 0; j <= top; j++)
            printf("%c", convert(stack[j]));
        printf("\t\t");

        for (j = i; ip[j] != -1; j++)
            printf("%c", convert(ip[j]));
        printf("\n");

        a = stack[top];
        b = ip[i];

        if (a == b && a == 10) {
            printf("\nSUCCESS: INPUT STRING PARSED SUCCESSFULLY.\n");
            break;
        } else if (a == b) {
            pop();
            i++;
        } else if (a < 5 && b >= 0 && b <= 10 && m[a][b] != 0) {
            t = m[a][b];
            pop();
            if (t != -2) {
                while (t > 0) {
                    push(t % 10);
                    t /= 10;
                }
            }
        } else {#include<stdio.h>
#define TOGETHER 8

int main(void)
{
    int i = 0;
    int entries = 50;
    int repeat;
    int left = 0;
    
    repeat = (entries / TOGETHER);
    left = (entries % TOGETHER);
    
    while (repeat--) {
        printf("process(%d)\n", i);
        printf("process(%d)\n", i + 1);
        printf("process(%d)\n", i + 2);
        printf("process(%d)\n", i + 3);
        printf("process(%d)\n", i + 4);
        printf("process(%d)\n", i + 5);
        printf("process(%d)\n", i + 6);
        printf("process(%d)\n", i + 7);
        i += TOGETHER;
    }
    
    switch (left) {
        case 7: printf("process(%d)\n", i + 6);
        case 6: printf("process(%d)\n", i + 5);
        case 5: printf("process(%d)\n", i + 4);
        case 4: printf("process(%d)\n", i + 3);
        case 3: printf("process(%d)\n", i + 2);
        case 2: printf("process(%d)\n", i + 1);
        case 1: printf("process(%d)\n", i);
        case 0: break;
    }
    
    return 0;
}

constant propagation:

#include<stdio.h>
#include<string.h>
#include<ctype.h>
#include<stdlib.h>

void input();
void output();
void change(int p, char *res);
void constant();

struct expr {
    char op[2], op1[5], op2[5], res[5];
    int flag;
} arr[10];

int n;

void main() {
    input();
    constant();
    output();
}

void input() {
    int i;
    printf("\nEnter the maximum number of expressions: ");
    scanf("%d", &n);
    printf("\nEnter the input (operation, operand1, operand2, result):\n");
    for(i = 0; i < n; i++) {
        scanf("%s", arr[i].op);
        scanf("%s", arr[i].op1);
        scanf("%s", arr[i].op2);
        scanf("%s", arr[i].res);
        arr[i].flag = 0;
    }
}

void constant() {
    int i;
    int op1, op2, res;
    char op, res1[5];

    for(i = 0; i < n; i++) {
        if(isdigit(arr[i].op1[0]) && isdigit(arr[i].op2[0]) || strcmp(arr[i].op, "=") == 0) {
            op1 = atoi(arr[i].op1);
            op2 = atoi(arr[i].op2);
            op = arr[i].op[0];
            
            switch(op) {
                case '+':
                    res = op1 + op2;
                    break;
                case '-':
                    res = op1 - op2;
                    break;
                case '*':
                    res = op1 * op2;
                    break;
                case '/':
                    res = op1 / op2;
                    break;
                case '=':
                    res = op1;
                    break;
                default:
                    res = 0;
                    break;
            }

            sprintf(res1, "%d", res);
            arr[i].flag = 1;
            change(i, res1);
        }
    }
}

void output() {
    int i;
    printf("\nOptimized code is:\n");
    for(i = 0; i < n; i++) {
        if(!arr[i].flag) {
            printf("%s %s %s %s\n", arr[i].op, arr[i].op1, arr[i].op2, arr[i].res);
        }
    }
}

void change(int p, char *res) {
    int i;
    for(i = p + 1; i < n; i++) {
        if(strcmp(arr[p].res, arr[i].op1) == 0) {
            strcpy(arr[i].op1, res);
        } else if(strcmp(arr[p].res, arr[i].op2) == 0) {
            strcpy(arr[i].op2, res);
        }
    }
}

            printf("\nERROR: INVALID STRING.\n");
            break;
        }
    }

    return 0;
}

LOOP UNROLLING:

#include<stdio.h>
#include<string.h>
#include<ctype.h>
#include<stdlib.h>

void input();
void output();
void change(int p, char *res);
void constant();

struct expr {
    char op[2], op1[5], op2[5], res[5];
    int flag;
} arr[10];

int n;

void main() {
    input();
    constant();
    output();
}

void input() {
    int i;
    printf("\nEnter the maximum number of expressions: ");
    scanf("%d", &n);
    printf("\nEnter the input (operation, operand1, operand2, result):\n");
    for(i = 0; i < n; i++) {
        scanf("%s", arr[i].op);
        scanf("%s", arr[i].op1);
        scanf("%s", arr[i].op2);
        scanf("%s", arr[i].res);
        arr[i].flag = 0;
    }
}

void constant() {
    int i;
    int op1, op2, res;
    char op, res1[5];

    for(i = 0; i < n; i++) {
        if(isdigit(arr[i].op1[0]) && isdigit(arr[i].op2[0]) || strcmp(arr[i].op, "=") == 0) {
            op1 = atoi(arr[i].op1);
            op2 = atoi(arr[i].op2);
            op = arr[i].op[0];
            
            switch(op) {
                case '+':
                    res = op1 + op2;
                    break;
                case '-':
                    res = op1 - op2;
                    break;
                case '*':
                    res = op1 * op2;
                    break;
                case '/':
                    res = op1 / op2;
                    break;
                case '=':
                    res = op1;
                    break;
                default:
                    res = 0;
                    break;
            }

            sprintf(res1, "%d", res);
            arr[i].flag = 1;
            change(i, res1);
        }
    }
}

void output() {
    int i;
    printf("\nOptimized code is:\n");
    for(i = 0; i < n; i++) {
        if(!arr[i].flag) {
            printf("%s %s %s %s\n", arr[i].op, arr[i].op1, arr[i].op2, arr[i].res);
        }
    }
}

void change(int p, char *res) {
    int i;
    for(i = p + 1; i < n; i++) {
        if(strcmp(arr[p].res, arr[i].op1) == 0) {
            strcpy(arr[i].op1, res);
        } else if(strcmp(arr[p].res, arr[i].op2) == 0) {
            strcpy(arr[i].op2, res);
        }
    }
}



