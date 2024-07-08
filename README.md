#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char *argv[]) {
    if (argc != 2) {
        printf("Usage: cd [directory]\n");
        return 1;
    }

    char *dir = argv[1];
    if (strcmp(dir, "-") == 0) {
        if (chdir(getenv("OLDPWD")) != 0) {
            perror("cd -");
            return 1;
        }
    } else {
        if (chdir(dir) != 0) {
            perror("cd");
            return 1;
        }
    }

    return 0;
}
