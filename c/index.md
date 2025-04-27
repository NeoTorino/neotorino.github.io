# C


To compile run:
```Shell
gcc list_files.c -o list_files
./list_files /path/to/directory
```

list_files.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <dirent.h>
#include <string.h>

int main(int argc, char *argv[]) {
    struct dirent *de;  // Pointer for directory entry
    DIR *dr;

    if (argc != 2) {
        printf("Usage: %s <directory_path>\n", argv[0]);
        return 1;
    }

    // Open the directory
    dr = opendir(argv[1]);
    if (dr == NULL) {
        perror("Could not open directory");
        return 1;
    }

    printf("Listing files in directory: %s\n", argv[1]);

    while ((de = readdir(dr)) != NULL) {
        printf("%s\n", de->d_name);
    }

    closedir(dr);
    return 0;
}
```

