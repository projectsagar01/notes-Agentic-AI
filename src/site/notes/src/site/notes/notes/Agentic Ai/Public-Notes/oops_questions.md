---
{"dg-publish":true,"permalink":"/src/site/notes/notes/agentic-ai/public-notes/oops-questions/","dg-note-properties":{}}
---

// /******************************************************************************

//                               Online C++ Compiler.
//               Code, Compile, Run and Debug C++ program online.
// Write your code in this editor and press "Run" button to compile and execute it.

// *******************************************************************************/

// #include <bits/stdc++.h>
// using namespace std;

// class Student{
//     private:
//         string name;
//         int marks;
//     public:
//         void setData(string naam, int num){
//             name = naam;
//             marks = num;
//         }
//         void display(){
//             cout << name << " " <<  "got" << " " << marks << endl;
//         }
//         void isPassed(){
//             if(marks>=40){
//                 cout << "True" << endl;
//             }
//             else{
//                 cout << "not passed" << endl;
//             }
//         }
// };

// int main()
// {
//   Student s1;
//   s1.setData("prem", 30);
//   s1.display();
//   s1.isPassed();
//     return 0;
// }



// #include <bits/stdc++.h>
// using namespace std;

// class Transfer{
        
//         private:
//             int taccountNumber;
//             string taccountHolder;
//             double tbalance;
     
        
// }

// class BankAccount : public Transfer{
//     private:
//         int accountNumber;
//         string accountHolder;
//         double balance;
//     public:
//         void deposit(int deposite){
//               deposite = d;                
//         }
//         void withdraw (int withdraw) {
//             if(withdraw<balance){
//                 cout << "Not enough balance for" <<
//                 "try minimum account for withdraw"<< endl;
//             }else{
//                 cout << " Amount i withdrawl successfully" << endl;
//             }
//         }
//         void transfer (taccountNumber,taccountHolder, tbalance) {
//              if(transfer<balance){
//             cout << "Money transfer to : --" << endl;
//             cout << "Account Number " << taccountNumber << endl;
//             cout << "transfer Balance" << tbalace << endl;
//              }else{
//                  cout << " You don't have enough money to transfer"<< endl;
//              }
//         }
        
//         void displayAccount(){
//             cout << " account number " << accountNumber;
//             cout << "account holder name " << accountHolder;
//             cout << "account balance" << balance;
//         }
// };

// int main () {
    
    
//     return 0;
// }

#include <iostream>
using namespace std;

class BankAccount {
private:
    int accountNumber;
    string accountHolder;
    double balance;

public:

    // Constructor
    BankAccount(int accNo, string holder, double initialBalance) {
        accountNumber = accNo;
        accountHolder = holder;

        if (initialBalance >= 0)
            balance = initialBalance;
        else
            balance = 0;
    }

    void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            cout << "Deposit successful!" << endl;
        } else {
            cout << "Invalid deposit!" << endl;
        }
    }

    void withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            cout << "Withdrawal successful!" << endl;
        } else {
            cout << "Invalid withdrawal!" << endl;
        }
    }

    void transfer(BankAccount &receiver, double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            receiver.balance += amount;

            cout << "Transfer successful!" << endl;
        } else {
            cout << "Transfer failed!" << endl;
        }
    }

    void displayAccount() {
        cout << "\nAccount Number: " << accountNumber << endl;
        cout << "Account Holder: " << accountHolder << endl;
        cout << "Balance: " << balance << endl;
    }
};

int main() {

    int accNo;
    string name;
    double balance;

    // Account 1
    cout << "Enter Account 1 number: ";
    cin >> accNo;

    cout << "Enter Account 1 holder name: ";
    cin >> name;

    cout << "Enter Account 1 balance: ";
    cin >> balance;

    BankAccount a(accNo, name, balance);


    // Account 2
    cout << "\nEnter Account 2 number: ";
    cin >> accNo;

    cout << "Enter Account 2 holder name: ";
    cin >> name;

    cout << "Enter Account 2 balance: ";
    cin >> balance;

    BankAccount b(accNo, name, balance);


    // Account 3
    cout << "\nEnter Account 3 number: ";
    cin >> accNo;

    cout << "Enter Account 3 holder name: ";
    cin >> name;

    cout << "Enter Account 3 balance: ";
    cin >> balance;

    BankAccount c(accNo, name, balance);


    // Deposit
    double amount;

    cout << "\nEnter deposit amount for Account 1: ";
    cin >> amount;
    a.deposit(amount);


    // Withdrawal
    cout << "Enter withdrawal amount from Account 1: ";
    cin >> amount;
    a.withdraw(amount);


    // Transfer
    cout << "Enter amount to transfer from Account 1 to Account 2: ";
    cin >> amount;
    a.transfer(b, amount);


    // Failed transfer
    cout << "Enter amount to transfer from Account 2 to Account 3: ";
    cin >> amount;
    b.transfer(c, amount);


    // Display accounts
    cout << "\n----- ACCOUNT DETAILS -----" << endl;

    a.displayAccount();
    b.displayAccount();
    c.displayAccount();

    return 0;
}






































