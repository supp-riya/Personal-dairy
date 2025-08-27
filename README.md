#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

#define FILENAME "diary.txt"

typedef struct {
    char datetime[30];
    char text[500];
} DiaryEntry;

// Function to get current date and time
void getDateTime(char *buffer) {
    time_t t;
    struct tm *tm_info;
    time(&t);
    tm_info = localtime(&t);
    strftime(buffer, 30, "%Y-%m-%d %H:%M:%S", tm_info);
}

// Function to add a diary entry
void addEntry() {
    FILE *fp = fopen(FILENAME, "a");
    if (!fp) {
        printf("Error opening file!\n");
        return;
    }
    DiaryEntry entry;
    getDateTime(entry.datetime);
    printf("Write your diary entry:\n");
    getchar(); // clear buffer
    fgets(entry.text, sizeof(entry.text), stdin);
    entry.text[strcspn(entry.text, "\n")] = '\0'; // remove newline
    fprintf(fp, "%s|%s\n", entry.datetime, entry.text);
    fclose(fp);
    printf("Entry added successfully!\n");
}

// Function to view all entries
void viewEntries() {
    FILE *fp = fopen(FILENAME, "r");
    if (!fp) {
        printf("No entries found.\n");
        return;
    }
    char line[600];
    int count = 1;
    printf("\n------ Diary Entries ------\n");
    while (fgets(line, sizeof(line), fp)) {
        char *datetime = strtok(line, "|");
        char *text = strtok(NULL, "\n");
        printf("%d. [%s]\n   %s\n", count++, datetime, text);
    }
    fclose(fp);
    printf("---------------------------\n");
}

// Function to edit an entry
void editEntry() {
    FILE *fp = fopen(FILENAME, "r");
    if (!fp) {
        printf("No entries to edit.\n");
        return;
    }

    // Load all entries
    DiaryEntry entries[100];
    int count = 0;
    char line[600];
    while (fgets(line, sizeof(line), fp)) {
        strcpy(entries[count].datetime, strtok(line, "|"));
        strcpy(entries[count].text, strtok(NULL, "\n"));
        count++;
    }
    fclose(fp);

    viewEntries();
    int choice;
    printf("Enter entry number to edit: ");
    scanf("%d", &choice);
    getchar(); // clear input buffer

    if (choice < 1 || choice > count) {
        printf("Invalid choice.\n");
        return;
    }

    printf("Enter new text:\n");
    fgets(entries[choice - 1].text, sizeof(entries[choice - 1].text), stdin);
    entries[choice - 1].text[strcspn(entries[choice - 1].text, "\n")] = '\0';

    fp = fopen(FILENAME, "w");
    for (int i = 0; i < count; i++) {
        fprintf(fp, "%s|%s\n", entries[i].datetime, entries[i].text);
    }
    fclose(fp);
    printf("Entry updated successfully!\n");
}

// Function to delete an entry
void deleteEntry() {
    FILE *fp = fopen(FILENAME, "r");
    if (!fp) {
        printf("No entries to delete.\n");
        return;
    }

    DiaryEntry entries[100];
    int count = 0;
    char line[600];
    while (fgets(line, sizeof(line), fp)) {
        strcpy(entries[count].datetime, strtok(line, "|"));
        strcpy(entries[count].text, strtok(NULL, "\n"));
        count++;
    }
    fclose(fp);

    viewEntries();
    int choice;
    printf("Enter entry number to delete: ");
    scanf("%d", &choice);

    if (choice < 1 || choice > count) {
        printf("Invalid choice.\n");
        return;
    }

    fp = fopen(FILENAME, "w");
    for (int i = 0; i < count; i++) {
        if (i != choice - 1) {
            fprintf(fp, "%s|%s\n", entries[i].datetime, entries[i].text);
        }
    }
    fclose(fp);
    printf("Entry deleted successfully!\n");
}

// Main menu
int main() {
    int choice;
    while (1) {
        printf("\n==== Personal Diary System ====\n");
        printf("1. Add Entry\n");
        printf("2. View Entries\n");
        printf("3. Edit Entry\n");
        printf("4. Delete Entry\n");
        printf("5. Exit\n");
        printf("Enter choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1: addEntry(); break;
            case 2: viewEntries(); break;
            case 3: editEntry(); break;
            case 4: deleteEntry(); break;
            case 5: exit(0);
            default: printf("Invalid choice!\n");
        }
    }
    return 0;
}
