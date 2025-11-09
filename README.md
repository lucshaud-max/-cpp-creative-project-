#include <iostream>
#include <fstream>
#include <string>
#include <cstdlib>
#include <ctime>
#include <limits>

using namespace std;

// ===== Data Structure =====
struct PasswordRecord {
    string password;
    string rating;
    int length;
    string note;
};

// ===== Function Prototypes =====
void showMenu();
int getChoice();
string generatePassword(int length, bool useSymbols);
string evaluateStrength(const string& password);
void simulateBruteForce(const string& password);
bool saveRecordToFile(const PasswordRecord& rec, const string& filename);
int loadRecordsFromFile(PasswordRecord records[], int maxSize, const string& filename);
void printReport(const PasswordRecord& rec);
void printReport(const PasswordRecord records[], int count);

// ===== Function Definitions =====
void showMenu() {
    cout << "\n=== CyberGuard Password Lab ===\n";
    cout << "1. Generate a strong password\n";
    cout << "2. Evaluate a password's strength\n";
    cout << "3. Simulate brute-force on a simple password\n";
    cout << "4. View password history from file\n";
    cout << "5. Exit\n";
}

int getChoice() {
    int choice;
    cout << "Enter your choice (1-5): ";
    while (!(cin >> choice) || choice < 1 || choice > 5) {
        cout << "Invalid choice. Enter 1-5: ";
        cin.clear();
        cin.ignore(numeric_limits<streamsize>::max(), '\n');
    }
    cin.ignore(numeric_limits<streamsize>::max(), '\n');
    return choice;
}

string generatePassword(int length, bool useSymbols) {
    const string letters = "abcdefghijklmnopqrstuvwxyz";
    const string digits  = "0123456789";
    const string upper   = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
    const string symbols = "!@#$%^&*()-_=+[]{}<>?";
    string pool = letters + digits + upper;
    if (useSymbols) pool += symbols;

    string result;
    result.reserve(length);
    for (int i = 0; i < length; ++i)
        result.push_back(pool[rand() % pool.size()]);
    return result;
}

string evaluateStrength(const string& password) {
    bool hasLower=false, hasUpper=false, hasDigit=false, hasSymbol=false;
    for (char c : password) {
        if (islower((unsigned char)c)) hasLower=true;
        else if (isupper((unsigned char)c)) hasUpper=true;
        else if (isdigit((unsigned char)c)) hasDigit=true;
        else hasSymbol=true;
    }
    int score=0;
    if (password.length()>=8) score++;
    if (password.length()>=12) score++;
    if (hasLower && hasUpper) score++;
    if (hasDigit) score++;
    if (hasSymbol) score++;

    if (score<=2) return "Weak";
    else if (score<=4) return "Medium";
    else return "Strong";
}

void simulateBruteForce(const string& password) {
    cout << "\nSimulating brute-force on password: " << password << endl;
    cout << "(Numeric-only simulation for learning)\n";

    bool numericOnly=true;
    for (char c : password)
        if (!isdigit((unsigned char)c)) numericOnly=false;

    if (!numericOnly || password.length()>4) {
        cout << "Only numeric passwords up to 4 digits are allowed.\n";
        return;
    }

    int attempts=0, target=stoi(password);
    for (int guess=0; guess<=9999; ++guess) {
        attempts++;
        if (guess==target) {
            cout << "Password cracked! Guess: " << guess 
                 << "  Attempts: " << attempts << endl;
            break;
        }
    }
}

// ===== File Handling =====
bool saveRecordToFile(const PasswordRecord& rec, const string& filename) {
    ofstream out(filename, ios::app);
    if (!out) {
        cout << "File write error!\n";
        return false;
    }
    out << rec.password << "," << rec.rating << "," 
        << rec.length << "," << rec.note << "\n";
    return true;
}

int loadRecordsFromFile(PasswordRecord records[], int maxSize, const string& filename) {
    ifstream in(filename);
    if (!in) return 0;
    int count=0; string line;
    while (getline(in,line) && count<maxSize) {
        size_t p1=line.find(','); size_t p2=line.find(',',p1+1);
        size_t p3=line.find(',',p2+1);
        if (p1==string::npos||p2==string::npos||p3==string::npos) continue;
        PasswordRecord r;
        r.password=line.substr(0,p1);
        r.rating=line.substr(p1+1,p2-p1-1);
        r.length=stoi(line.substr(p2+1,p3-p2-1));
        r.note=line.substr(p3+1);
        records[count++]=r;
    }
    return count;
}

// ===== Overloaded Report Functions =====
void printReport(const PasswordRecord& rec) {
    cout << "Password: " << rec.password << "\n"
         << "Rating:   " << rec.rating << "\n"
         << "Length:   " << rec.length << "\n"
         << "Note:     " << rec.note << "\n\n";
}

void printReport(const PasswordRecord records[], int count) {
    cout << "\n=== Password History (" << count << " records) ===\n";
    for (int i=0; i<count; ++i) {
        cout << "Record #" << (i+1) << ":\n";
        printReport(records[i]); // calls the single-record overload
    }
}

// ===== main =====
int main() {
    srand(static_cast<unsigned int>(time(nullptr)));

    PasswordRecord records[100];
    int recordCount = 0;
    const string filename = "password_log.txt";
    bool running = true;

    while (running) {
        showMenu();
        int choice = getChoice();

        if (choice == 1) {
            int length; char sym;
            cout << "Enter password length (8-32): ";
            while (!(cin >> length) || length<8 || length>32) {
                cout << "Invalid. Enter 8-32: ";
                cin.clear();
                cin.ignore(numeric_limits<streamsize>::max(), '\n');
            }
            cout << "Include symbols? (y/n): "; cin >> sym;
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
            bool useSymbols = (sym=='y'||sym=='Y');

            string pwd = generatePassword(length,useSymbols);
            string rate = evaluateStrength(pwd);
            cout << "Generated password: " << pwd 
                 << " (" << rate << ")\n";

            if (recordCount<100) {
                records[recordCount] = {pwd, rate, length, "Generated"};
                saveRecordToFile(records[recordCount], filename);
                recordCount++;
            }

        } else if (choice == 2) {
            string pwd;
            cout << "Enter password to evaluate: ";
            getline(cin, pwd);
            string rate = evaluateStrength(pwd);
            cout << "Strength: " << rate << "\n";
            if (recordCount<100) {
                records[recordCount] = {pwd, rate, (int)pwd.size(), "User tested"};
                saveRecordToFile(records[recordCount], filename);
                recordCount++;
            }

        } else if (choice == 3) {
            string pwd;
            cout << "Enter numeric password (max 4 digits): ";
            getline(cin, pwd);
            simulateBruteForce(pwd);

        } else if (choice == 4) {
            int n = loadRecordsFromFile(records, 100, filename);
            if (n>0) printReport(records, n);
            else cout << "No records found.\n";

        } else if (choice == 5) {
            cout << "Exiting CyberGuard. Stay secure!\n";
            running = false;
        }
    }
    return 0;
}
