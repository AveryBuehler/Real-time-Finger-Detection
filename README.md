<div align="center">
  <h1>Real-time Finger Detection</h1>
</div>

<div align="center">
  <strong>Written entirely in Python using OpenCV</strong>
</div>  

<div align="center">
  <img src="https://media.giphy.com/media/XeMmEZSzLK0mLwzhPG/giphy.gif" alt="Real-time finger detection example">
</div>
  
<div align="center">  
  <sub>Made for CS390 - Machine Perception</sub>
</div>

## Table of Contents
- [Background](#background)
- [Requirements](#requirements)
- [Features](#features)
- [Python Code Samples](#c-code-samples)
- [Future Improvements](#future-improvements)

## Background
The purpose of this project was to create a real-time finger detection program using Python and OpenCV. To accomplish this, we made use of **convex hulls** - a convex closure of a set *X* of points in a Euclidean space that is the smallest convex set that contains *X*. 

The image below shows a high-level example of how convex hulls work. For our project, the convex hull was used to segment the hand and find the fingers by looking into extremities which touches the edge of the convex hull.
<div align="center">
  <img src="https://miro.medium.com/max/1354/1*F4IUmOJbbLMJiTgHxpoc7Q.png" alt="Convex hull example">
</div>
  
## Requirements
1. [Anaconda 2019.03](https://www.anaconda.com/distribution/) (Python 3.7 version)
2. [OpenCV](https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_tutorials.html)  
	```conda install -c conda-forge opencv```
3. [imutils](https://github.com/jrosebr1/imutils)   
	```conda install -c pjamesjoyce imutils```
## Features
- **Finger derection** in real-time by using a webcam
- **Visually displays** various interesting aspects (palm center, extremities, etc.)
- In-depth use of **OpenCV**
- **Region of interest** (ROI) detection
- **Finger-frame summation** to account for errors
- **Skin detection** by using a weighted background algorithm

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
    

