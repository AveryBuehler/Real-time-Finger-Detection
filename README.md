<div align="center">
  <h1>Real-time Finger Detection With Python and OpenCV </h1>
</div>

<div align="center">
  <strong>An implementation of the Caesar cipher written entirely in C</strong>
</div>  

<div align="center">
  <img src="https://cdncontribute.geeksforgeeks.org/wp-content/uploads/ceaserCipher.png" alt="Caesar cipher example">
</div>
  
<div align="center">  
  <sub>Made for CS455 - System Programming</sub>
</div>

## Table of Contents
- [Background](#background)
- [Project Description](#project-description)
- [Features](#features)
- [C Code Samples](#c-code-samples)
- [Future Improvements](#future-improvements)

## Background
  <sub><a href="https://en.wikipedia.org/wiki/Caesar_cipher" target="_blank">Information obtained from Wikipedia</a></sub>
  >"The Caesar cipher is one of the simplest and most widely known encryption techniques. It is a type of substitution cipher in which each letter in the plaintext is replaced by a letter some fixed number of positions down the alphabet. For example, with a left shift of `3`, `D` would be replaced by `A`, `E` would become `B`, and so on. The method is named after Julius Caesar, who used it in his private correspondence."
  
## Project Description
<sub>Information from CS455 - System Programming</sub>
1. Write a program that encrypts a message using a Caesar cipher. The user will enter the plaintext (message to be encrypted) and the shift amount:

    ```
    Enter message to be encrypted: Go ahead, make my day.
    Enter shift amount (1-25): 3
    Encrypted message: Jr dkhdg, pbnh gdb.
    ```
      
	Notice that the program can decrypt a message if the user enters 26 minus the original shift amount:

	  ```
	  Enter message to be encrypted: Jr dkhdg, pbnh gdb.
	  Enter shift amount (1-25): 23
	  Encrypted message: Go ahead, make my day.
	  ```
      
   **Assumptions**
   - You may assume that the message does not exceed 80 characters. Characters other than letters are left unchanged. 
   - Lowercase letters should remain lowercase when encrypted; uppercase letters should remain uppercase as well.
  
2. Modify the above program so that the program prompts the user to enter the name of a file containing the message to be encrypted:
  
    ```
    Enter name of file to be encrypted: message.txt
    Enter shift amount (1-25): 3
    ```
  
  	The program should write the encrypted message to the same file path but with an added extension of `.enc`. In the example above, the original file name is `message.txt`, so the encrypted message will be stored as `message.txt.enc`.  
      
   **Assumptions**
   - There is no limit on the size of the file to be encrypted or the length of the lines in the file. 

## Features
- **Dynamic memory allocation** based on file size and plaintext
- **Cross-platform** between Windows and Linux for printing the working directory
- Wide use of **pointers**
- **Error handling**

## C Code Samples

### main.c
This is the driver class.

### encrypt.c
This is used for keyboard I/O.
```c
#define MAX_CHAR 81

void read_from_keyboard(void);

void read_from_keyboard(void) {
    char *message = malloc(sizeof(char) * MAX_CHAR);
    if(message == NULL) {
        printf("Insufficient memory.\n");
        exit(EXIT_FAILURE);
    }
    printf("Enter a message: ");
    fgets(message, MAX_CHAR, stdin);
    if((strlen(message) > 0) && (message[strlen(message) -1] == '\n')) {
        message[strlen(message) - 1] = '\0';
    }
    char *cipher_text = encrypt_text(message);
    printf("Encrypted message: %s\n\n", cipher_text);
    free(message);
}
```

### encrypt_text.c
This is used for encrypting text as per Project Description #1.
```c
char *encrypt_text(char *str);

char *encrypt_text(char *str) {
    char *shift_char = malloc(sizeof(shift_char));
    int shift_num = 0;
    do {
        printf("Enter shift amount (1-25): ");
        fgets(shift_char, sizeof(shift_num), stdin);
        shift_num = atoi(shift_char);
        if(shift_num < 1 || shift_num > 25) {
            printf("Please enter a valid shift amount.\n");
        }
    }while(shift_num <1 || shift_num > 25);
    for(int i = 0; str[i] != '\0'; i++) {
        if((str[i] >= 65 && str[i] <= 90) || (str[i] >= 97 && str[i] <= 122)) {
            if(str[i] >= 97) {
                str[i] = ((str[i] - 97) + shift_num)%26 + 97;
            } else {
                str[i] = ((str[i] - 65) + shift_num)%26 + 65;
            }
        }
    }
    return str;
}
```

### encrypt_text.h
This is used for the encrypt_text() declaration.

### encrypt_file.c
This is used for encrypting files as per Project Description #2. Works on Windows & Linux operating systems.
```c
#if defined(_WIN32) || defined(_WIN64)
    #define PLATFORM_NAME "windows"
    #define CWD_PLATFORM (cwd = _getcwd(NULL, 0)
    #include <direct.h>
#endif
#if defined(__linux__)
    #define PLATFORM_NAME "linux"
    #define CWD_PATH_MAX PATH_MAX
    #define CWD_PLATFORM (cwd = getcwd(NULL, 0)
    #include <unistd.h>
#endif
#define MAX_FILE_SIZE 512

void read_from_file(void);

void read_from_file(void) {
    char file_name[MAX_FILE_SIZE];
    char *cwd;
    char *text;
    FILE *file;
    long file_size;
    size_t result;
    printf("Enter the name of the file to be encrypted: ");
    fgets(file_name, MAX_FILE_SIZE, stdin);
    file_name[strcspn(file_name, "\n")] = 0;
    if((file = fopen(file_name, "rb")) == NULL) {
        perror("Cannot open file");
        exit(EXIT_FAILURE);
    }
    fseek(file, 0L, SEEK_END);
    file_size = ftell(file);
    rewind(file);
    text = malloc(file_size + 1);
    text[file_size] = '\0';
    if(text == NULL) {
        perror("Insufficient memory");
        exit(EXIT_FAILURE);
    }
    result = fread(text, 1, file_size, file);
    if(result != file_size) {
        perror("Error reading file");
        exit(EXIT_FAILURE);
    }
    char *cipher_text = encrypt_text(text);
    char *new_file = strcat(file_name, ".enc");
    file = fopen(new_file, "wb");
    fwrite(cipher_text, sizeof(char), strlen(cipher_text), file);
    fclose(file);
    if(CWD_PLATFORM) != NULL) {
        if(PLATFORM_NAME == "windows") {
            printf("Saved encrypted text to: %s\\%s\n", cwd, new_file);
        }
        else if(PLATFORM_NAME == "linux") {
            printf("Saved encrypted text to: %s/%s\n", cwd, new_file);
        }
    } else {
        perror("CWD error");
        exit(EXIT_FAILURE);
    }
    free(text);
}
```

## Future Improvements
- [ ] Add compatability with all operating systems
- [ ] Add a GUI with C++

## License
[MIT](https://tldrlegal.com/license/mit-license)
    

