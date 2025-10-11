#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

#define MAX_CMD_LEN 1024
#define MAX_ARGS 100

// Function to perform search
void search_command(char **args, int arg_count) {
    if (arg_count < 3) {
        fprintf(stderr, "Usage:\n");
        fprintf(stderr, "  search a filename pattern\n");
        fprintf(stderr, "  search c filename pattern\n");
        return;
    }

    char *mode = args[1];
    char *filename;
    char *pattern;

    if (strcmp(mode, "a") == 0 && arg_count == 4) {
        filename = args[2];
        pattern = args[3];

        FILE *file = fopen(filename, "r");
        if (!file) {
            perror("File open failed");
            return;
        }

        char line[1024];
        int line_number = 1;

        while (fgets(line, sizeof(line), file)) {
            if (strstr(line, pattern)) {
                printf("Line %d: %s", line_number, line);
            }
            line_number++;
        }

        fclose(file);

    } else if (strcmp(mode, "c") == 0 && arg_count == 4) {
        filename = args[2];
        pattern = args[3];

        FILE *file = fopen(filename, "r");
        if (!file) {
            perror("File open failed");
            return;
        }

        char line[1024];
        int count = 0;

        while (fgets(line, sizeof(line), file)) {
            char *pos = line;
            while ((pos = strstr(pos, pattern)) != NULL) {
                count++;
                pos += strlen(pattern);  // Move past the last found pattern
            }
        }

        printf("Total occurrences of \"%s\": %d\n", pattern, count);
        fclose(file);
    } else {
        fprintf(stderr, "Invalid search command format.\n");
    }
}

int main() {
    char input[MAX_CMD_LEN];
    char *args[MAX_ARGS];

    while (1) {
        printf("myshell$ ");
        fflush(stdout);

        // Read input
        if (!fgets(input, sizeof(input), stdin)) {
            break;  // EOF or error
        }

        // Remove trailing newline
        input[strcspn(input, "\n")] = 0;

        // Skip empty input
        if (strlen(input) == 0) continue;

        // Tokenize input
        int arg_count = 0;
        char *token = strtok(input, " ");
        while (token != NULL && arg_count < MAX_ARGS - 1) {
            args[arg_count++] = token;
            token = strtok(NULL, " ");
        }
        args[arg_count] = NULL;

        if (strcmp(args[0], "exit") == 0) {
            break;
        }

        if (strcmp(args[0], "search") == 0) {
            search_command(args, arg_count);
            continue;
        }

        // External command
        pid_t pid = fork();
        if (pid < 0) {
            perror("Fork failed");
        } else if (pid == 0) {
            execvp(args[0], args);
            perror("Execution failed");
            exit(EXIT_FAILURE);
        } else {
            wait(NULL);  // Wait for child
        }
    }

    return 0;
}

