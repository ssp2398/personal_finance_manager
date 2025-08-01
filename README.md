# personal_finance_manager

This project is a web-based personal finance management tool built with Java (Spring Boot), JavaScript/CSS, and MySQL. It helps users track income, expenses, and manage budgets effectively.

# Core Features

1. User Authentication

    User registration and secure login

    Session/token-based authentication


2. Transaction Management

     Add, update, and delete income or expense entries

     Assign categories like Rent, Food, Salary, Utilities, etc.

     Support for date-based filtering


3. Analytics Dashboard

      Monthly and category-wise breakdown

      Visual reports using charts (e.g., Pie, Bar – via Chart.js)

      Transaction summaries and trends

## Coming Soon or Planned Features
The following features are planned for future update

** Budgeting Assistant (AI-powered) **

    Smart suggestions based on historical spending

    Warnings for potential overspending

    Monthly savings recommendations

# Database (SQL)

Database Schema Overview

The project uses MySQL as the database. Below is an overview of the key tables and their purpose:

user – Stores user credentials and profile data.

account – Tracks different user accounts (e.g., bank accounts, wallets) with balances and liabilities.

income – Records all income transactions along with source, amount, and date.

expense – Logs user expenses with categories, amounts, remarks, and timestamps.

income_source – Manages different sources of income per user (e.g., Salary, Freelance).

expense_category – Allows users to define and manage custom expense categories.

transaction – Auto-generated logs of every income/expense entry for auditing and summaries.

budget – Lets users define monthly spending limits per category.

target_amount – Users can set personal savings or financial goals.


# Trigger-Based Automation

Automatically logs income and expense transactions into the transaction table using MySQL triggers.

A deletion trigger ensures that removing an account also cleans up related income, expense, and transaction records.

Here is SQL  Database schema :


    create database pfm;
    
    use pfm;

    CREATE TABLE user (
             user_id INT AUTO_INCREMENT PRIMARY KEY,
             name VARCHAR(100),
             username VARCHAR(50) UNIQUE,
             password VARCHAR(100)
             );

            
    CREATE TABLE account (
             account_id INT AUTO_INCREMENT PRIMARY KEY,
             user_id INT,
             account_type VARCHAR(50),
             balance DECIMAL(10, 2),
             liabilities DECIMAL(10, 2),
             FOREIGN KEY (user_id) REFERENCES user(user_id)
             );
            

    CREATE TABLE income_source (
              source_id INT AUTO_INCREMENT PRIMARY KEY,
              user_id INT,
              source_name VARCHAR(100),
              FOREIGN KEY (user_id) REFERENCES user(user_id)
              );
              

    CREATE TABLE income (
              income_id INT AUTO_INCREMENT PRIMARY KEY,
              user_id INT,
              account_id INT,
              income_date DATE,
              income_source VARCHAR(100),
              amount DECIMAL(10, 2),
              FOREIGN KEY (user_id) REFERENCES user(user_id),
              FOREIGN KEY (account_id) REFERENCES account(account_id)
              );
              

    CREATE TABLE expense_category (
             category_id INT AUTO_INCREMENT PRIMARY KEY,
          	 user_id INT,
             category_name VARCHAR(100),
             FOREIGN KEY (user_id) REFERENCES user(user_id)
             );


     CREATE TABLE expense (
	          expense_id INT AUTO_INCREMENT PRIMARY KEY,
              user_id INT,
              account_id INT,
              expense_date DATE,
              expense_category VARCHAR(100),
              remark VARCHAR(100),
              amount DECIMAL(10, 2),
              FOREIGN KEY (user_id) REFERENCES user(user_id),
              FOREIGN KEY (account_id) REFERENCES account(account_id)
              );
              


    delimiter //
    
          create trigger beforeAccountdelete  before delete on account for each row

    begin
	      if OLD.account_id is not null THEN
		       delete from expense where account_id = OLD.account_id;
	               delete from income where account_id = OLD.account_id;
                   delete from transaction where account_id = OLD.account_id;
	               END IF;
             end;
           
             //
             

    delimiter ;
    


    CREATE TABLE transaction (
            transaction_id INT AUTO_INCREMENT PRIMARY KEY,
            account_id INT,
            type varchar(10),
            amount decimal(10,2),
	        statement VARCHAR(255),
            time TIMESTAMP,
            foreign key (account_id) references account(account_id)
            );


     DELIMITER //

          CREATE TRIGGER afterIncomeInsert
          AFTER INSERT ON income
          FOR EACH ROW
          BEGIN
          
         INSERT INTO transaction(account_id, type, amount,statement, time) VALUES 
                    (NEW.account_id, 'Income',NEW.amount,CONCAT('Income recorded: ', NEW.income_source, ' - Amount: ', NEW.amount), CURRENT_TIMESTAMP);
         END;
         
         
              //


            CREATE TRIGGER afterExpenseInsert
            AFTER INSERT ON expense
            FOR EACH ROW
            BEGIN
            
            INSERT INTO transaction(account_id, type, amount, statement, time) VALUES 
                    (NEW.account_id, 'Expense',NEW.amount, CONCAT('Expense recorded: ', NEW.expense_category, ' - Amount: ', NEW.amount), CURRENT_TIMESTAMP);
             END;
             

                 //


     DELIMITER ;


    CREATE TABLE budget (
             budget_id INT AUTO_INCREMENT PRIMARY KEY,
             user_id INT,
             expense_category VARCHAR(100),
             amount DECIMAL(10, 2),
             FOREIGN KEY (user_id) REFERENCES user(user_id)
             );
             

    CREATE TABLE target_amount (
             target_id INT AUTO_INCREMENT PRIMARY KEY,
             user_id INT,
             amount DECIMAL(10, 2),
             FOREIGN KEY (user_id) REFERENCES user(user_id)
             );


