# EasyUnitConverter
Robust C++ tool for unit conversions (temp, currency, length, number bases) &amp; calculations (arithmetic, trig, log). User-friendly console with history &amp; error handling. Clone, compile with g++ main.cpp, run ./EasyUnitConverter. MIT License.
#include <iostream>
#include <iomanip>
#include <string>
#include <sstream>
#include <cmath>
#include <limits>
#include <vector>

using namespace std;

const string GREEN_COLOR = "\033[32m";
const string RED_COLOR = "\033[31m";
const string YELLOW_COLOR = "\033[33m";
const string CYAN_COLOR = "\033[36m";
const string BLUE_COLOR = "\033[34m";
const string RESET_COLOR = "\033[0m";

struct HistoryEntry {
    string type, input, result;
    HistoryEntry(string t, string i, string r) : type(t), input(i), result(r) {}
};

class Converter {
protected:
    double value;
public:
    Converter(double val = 0.0) : value(val) {}
    virtual ~Converter() = default;
    virtual double convert() const = 0;
    virtual string getType() const = 0;
    void displayResult(const string& input, const string& unitInfo) const {
        const int COL_WIDTH = 15;
        double result = convert();
        cout << fixed << setprecision(2);
        ostringstream oss;
        oss << fixed << setprecision(2) << result;
        string resultStr = oss.str();

        cout << "+" << string(COL_WIDTH, '-') << "+" << string(COL_WIDTH, '-') << "+" << string(COL_WIDTH, '-') << "+\n";
        cout << "|" << setw(COL_WIDTH) << left << BLUE_COLOR + "Input" + RESET_COLOR
             << "|" << setw(COL_WIDTH) << left << BLUE_COLOR + unitInfo + RESET_COLOR
             << "|" << setw(COL_WIDTH) << left << GREEN_COLOR + resultStr + RESET_COLOR << "|\n";
        cout << "+" << string(COL_WIDTH, '-') << "+" << string(COL_WIDTH, '-') << "+" << string(COL_WIDTH, '-') << "+\n";
    }
};

class TemperatureConverter : public Converter {
private:
    char fromUnit, toUnit;
public:
    TemperatureConverter(double temp, char from, char to)
        : Converter(temp), fromUnit(toupper(from)), toUnit(toupper(to)) {}

    double convert() const override {
        if (fromUnit == 'C' && toUnit == 'F') return value * 9.0 / 5.0 + 32.0;
        if (fromUnit == 'F' && toUnit == 'C') return (value - 32.0) * 5.0 / 9.0;
        if (fromUnit == toUnit) return value;
        throw runtime_error("Invalid temperature units (use C or F)");
    }
    string getType() const override { return "Temperature"; }
};

class NumberBaseConverter : public Converter {
private:
    string inputValue;
    char fromBase, toBase;

    string convertToBase(long long num, int base) const {
        if (num == 0) return "0";
        string digits = "0123456789ABCDEF";
        string result;
        while (num > 0) {
            result = digits[num % base] + result;
            num /= base;
        }
        return result;
    }

public:
    NumberBaseConverter(string val, char from, char to)
        : Converter(0), inputValue(val), fromBase(toupper(from)), toBase(toupper(to)) {
        try {
            if (fromBase == 'B') value = static_cast<double>(stoi(val, nullptr, 2)); // Binary
            else if (fromBase == 'D') value = stod(val); // Decimal
            else if (fromBase == 'O') value = static_cast<double>(stoi(val, nullptr, 8)); // Octal
            else if (fromBase == 'H') value = static_cast<double>(stoi(val, nullptr, 16)); // Hexadecimal
            else throw runtime_error("Invalid source base (use B, D, O, H)");
        } catch (const invalid_argument& e) {
            throw runtime_error("Invalid number format for the specified base");
        } catch (const out_of_range& e) {
            throw runtime_error("Number out of range for conversion");
        }
    }

    double convert() const override {
        long long intValue = static_cast<long long>(value);
        if (toBase == 'B') return stod(convertToBase(intValue, 2));
        if (toBase == 'D') return value;
        if (toBase == 'O') return stod(convertToBase(intValue, 8));
        if (toBase == 'H') return stod(convertToBase(intValue, 16));
        throw runtime_error("Invalid target base (use B, D, O, H)");
    }

    string getResultString() const {
        long long intValue = static_cast<long long>(value);
        if (toBase == 'B') return convertToBase(intValue, 2);
        if (toBase == 'D') return to_string(intValue);
        if (toBase == 'O') return convertToBase(intValue, 8);
        if (toBase == 'H') return convertToBase(intValue, 16);
        throw runtime_error("Invalid target base");
    }

    void displayResult(const string& input, const string& unitInfo) const {
        const int COL_WIDTH = 15;
        string result = getResultString();

        // Fixed line: Corrected to ensure no stray characters
        cout << "+" << string(COL_WIDTH, '-') << "+" << string(COL_WIDTH, '-') << "+" << string(COL_WIDTH, '-') << "+\n";
        cout << "|" << setw(COL_WIDTH) << left << BLUE_COLOR + "Input" + RESET_COLOR
             << "|" << setw(COL_WIDTH) << left << BLUE_COLOR + unitInfo + RESET_COLOR
             << "|" << setw(COL_WIDTH) << left << GREEN_COLOR + result + RESET_COLOR << "|\n";
        cout << "+" << string(COL_WIDTH, '-') << "+" << string(COL_WIDTH, '-') << "+" << string(COL_WIDTH, '-') << "+\n";
    }

    string getType() const override { return "Number Base"; }
};

class LogarithmicCalculator : public Converter {
private:
    char logType;
public:
    LogarithmicCalculator(double val, char type) : Converter(val), logType(toupper(type)) {}

    double convert() const override {
        if (value <= 0) throw runtime_error("Logarithm undefined for non-positive numbers");
        if (logType == 'L') return log10(value);  // Base 10
        if (logType == 'N') return log(value);    // Natural log (base e)
        if (logType == 'B') return log2(value);   // Base 2
        throw runtime_error("Invalid log type (use L, N, B)");
    }
    string getType() const override { return "Logarithm"; }
};

class CurrencyConverter : public Converter {
private:
    char fromCurrency, toCurrency;
    static constexpr double INR_TO_USD = 0.012;
    static constexpr double USD_TO_INR = 83.33;
    static constexpr double USD_TO_EUR = 0.92;
    static constexpr double USD_TO_GBP = 0.79;
    static constexpr double EUR_TO_USD = 1.09;
    static constexpr double GBP_TO_USD = 1.27;
public:
    CurrencyConverter(double amount, char from, char to)
        : Converter(amount), fromCurrency(toupper(from)), toCurrency(toupper(to)) {}

    double convert() const override {
        if (fromCurrency == 'I' && toCurrency == 'U') return value * INR_TO_USD;
        if (fromCurrency == 'U' && toCurrency == 'I') return value * USD_TO_INR;
        if (fromCurrency == 'U' && toCurrency == 'E') return value * USD_TO_EUR;
        if (fromCurrency == 'U' && toCurrency == 'G') return value * USD_TO_GBP;
        if (fromCurrency == 'E' && toCurrency == 'U') return value * EUR_TO_USD;
        if (fromCurrency == 'G' && toCurrency == 'U') return value * GBP_TO_USD;
        if (fromCurrency == toCurrency) return value;
        throw runtime_error("Invalid or unsupported currency (use I, U, E, G)");
    }
    string getType() const override { return "Currency"; }
};

class LengthConverter : public Converter {
private:
    char fromUnit, toUnit;
    static constexpr double M_TO_FT = 3.28084;
    static constexpr double FT_TO_M = 0.3048;
public:
    LengthConverter(double length, char from, char to)
        : Converter(length), fromUnit(toupper(from)), toUnit(toupper(to)) {}

    double convert() const override {
        if (fromUnit == 'M' && toUnit == 'F') return value * M_TO_FT;
        if (fromUnit == 'F' && toUnit == 'M') return value * FT_TO_M;
        if (fromUnit == toUnit) return value;
        throw runtime_error("Invalid length units (use M or F)");
    }
    string getType() const override { return "Length"; }
};

class Calculator {
private:
    double num1, num2;
    char operation;
public:
    Calculator(double n1, char op, double n2 = 0.0) : num1(n1), num2(n2), operation(toupper(op)) {}

    double calculate() const {
        switch (operation) {
            case '+': return num1 + num2;
            case '-': return num1 - num2;
            case '*': return num1 * num2;
            case '/':
                if (num2 == 0) throw runtime_error("Division by zero");
                return num1 / num2;
            case '^': return pow(num1, num2);
            case 'S': return sin(num1 * M_PI / 180.0);
            case 'C': return cos(num1 * M_PI / 180.0);
            case 'T':
                if (cos(num1 * M_PI / 180.0) == 0) throw runtime_error("Tan undefined");
                return tan(num1 * M_PI / 180.0);
            default: throw runtime_error("Invalid operation (use +, -, *, /, ^, S, C, T)");
        }
    }

    void displayResult() const {
        try {
            const int COL_WIDTH = 15;
            double result = calculate();
            cout << fixed << setprecision(2);
            ostringstream oss;
            oss << fixed << setprecision(2) << num1 << " " << operation << (operation == 'S' || operation == 'C' || operation == 'T' ? "" : " " + to_string(num2));
            string inputStr = oss.str();
            oss.str(""); oss << fixed << setprecision(2) << result;
            string resultStr = oss.str();

            cout << "+" << string(COL_WIDTH, '-') << "+" << string(COL_WIDTH, '-') << "+" << string(COL_WIDTH, '-') << "+\n";
            cout << "|" << setw(COL_WIDTH) << left << BLUE_COLOR + "Input" + RESET_COLOR
                 << "|" << setw(COL_WIDTH) << left << BLUE_COLOR + inputStr + RESET_COLOR
                 << "|" << setw(COL_WIDTH) << left << GREEN_COLOR + resultStr + RESET_COLOR << "|\n";
            cout << "+" << string(COL_WIDTH, '-') << "+" << string(COL_WIDTH, '-') << "+" << string(COL_WIDTH, '-') << "+\n";
        } catch (const runtime_error& e) {
            const int COL_WIDTH = 45;
            cout << "+" << string(COL_WIDTH, '-') << "+\n";
            cout << "|" << setw(COL_WIDTH) << left << RED_COLOR + string("Error: ") + e.what() + RESET_COLOR << "|\n";
            cout << "+" << string(COL_WIDTH, '-') << "+\n";
        }
    }
};

class Program {
private:
    vector<HistoryEntry> history;

    void showWelcome() const {
        const int COL_WIDTH = 40;
        cout << "+" << string(COL_WIDTH, '-') << "+\n";
        cout << "|" << setw(COL_WIDTH) << left << CYAN_COLOR + "Welcome to the Professional Converter!" + RESET_COLOR << "|\n";
        cout << "|" << setw(COL_WIDTH) << left << CYAN_COLOR + "Advanced conversion and calculation tool" + RESET_COLOR << "|\n";
        cout << "+" << string(COL_WIDTH, '-') << "+\n";
        // Adding group credits
        cout << "\n+" << string(COL_WIDTH, '-') << "+\n";
        cout << "|" << setw(COL_WIDTH) << left << YELLOW_COLOR + "Group Leader: Shravan Nadkarni, N-61" + RESET_COLOR << "|\n";
        cout << "|" << setw(COL_WIDTH) << left << YELLOW_COLOR + "Group Members:" + RESET_COLOR << "|\n";
        cout << "|" << setw(COL_WIDTH) << left << YELLOW_COLOR + "Parth Ghodke, N-20" + RESET_COLOR << "|\n";
        cout << "|" << setw(COL_WIDTH) << left << YELLOW_COLOR + "Prathamesh Chaumwal, I-31" + RESET_COLOR << "|\n";
        cout << "|" << setw(COL_WIDTH) << left << YELLOW_COLOR + "Gokul Krishnan A V, I-14" + RESET_COLOR << "|\n";
        cout << "|" << setw(COL_WIDTH) << left << YELLOW_COLOR + "Tapas Pandita, N-66" + RESET_COLOR << "|\n";
        cout << "|" << setw(COL_WIDTH) << left << YELLOW_COLOR + "Tejas Waghmare, N-72" + RESET_COLOR << "|\n";
        cout << "+" << string(COL_WIDTH, '-') << "+\n";
    }

    void clearInputBuffer() const {
        cin.clear();
        cin.ignore(numeric_limits<streamsize>::max(), '\n');
    }

    double getDoubleInput(const string& prompt) const {
        double input;
        cout << YELLOW_COLOR << prompt << RESET_COLOR;
        while (!(cin >> input)) {
            cout << RED_COLOR << "Invalid input. " << RESET_COLOR << YELLOW_COLOR << prompt << RESET_COLOR;
            clearInputBuffer();
        }
        return input;
    }

    char getCharInput(const string& prompt) const {
        char input;
        cout << YELLOW_COLOR << prompt << RESET_COLOR;
        cin >> input;
        clearInputBuffer();
        return input;
    }

    string getStringInput(const string& prompt) const {
        string input;
        cout << YELLOW_COLOR << prompt << RESET_COLOR;
        cin >> input;
        clearInputBuffer();
        return input;
    }

    void displayMenu() const {
        const int COL_WIDTH = 25;
        cout << "\n+" << string(COL_WIDTH, '-') << "+" << string(COL_WIDTH, '-') << "+\n";
        cout << "|" << setw(COL_WIDTH) << left << CYAN_COLOR + "Option" + RESET_COLOR
             << "|" << setw(COL_WIDTH) << left << CYAN_COLOR + "Description" + RESET_COLOR << "|\n";
        cout << "+" << string(COL_WIDTH, '-') << "+" << string(COL_WIDTH, '-') << "+\n";
        cout << "|" << setw(COL_WIDTH) << left << CYAN_COLOR + "1" + RESET_COLOR
             << "|" << setw(COL_WIDTH) << left << CYAN_COLOR + "Calculator" + RESET_COLOR << "|\n";
        cout << "|" << setw(COL_WIDTH) << left << CYAN_COLOR + "2" + RESET_COLOR
             << "|" << setw(COL_WIDTH) << left << CYAN_COLOR + "Temperature (C/F)" + RESET_COLOR << "|\n";
        cout << "|" << setw(COL_WIDTH) << left << CYAN_COLOR + "3" + RESET_COLOR
             << "|" << setw(COL_WIDTH) << left << CYAN_COLOR + "Number Base (B/D/O/H)" + RESET_COLOR << "|\n";
        cout << "|" << setw(COL_WIDTH) << left << CYAN_COLOR + "4" + RESET_COLOR
             << "|" << setw(COL_WIDTH) << left << CYAN_COLOR + "Logarithm (L/N/B)" + RESET_COLOR << "|\n";
        cout << "|" << setw(COL_WIDTH) << left << CYAN_COLOR + "5" + RESET_COLOR
             << "|" << setw(COL_WIDTH) << left << CYAN_COLOR + "Currency (I/U/E/G)" + RESET_COLOR << "|\n";
        cout << "|" << setw(COL_WIDTH) << left << CYAN_COLOR + "6" + RESET_COLOR
             << "|" << setw(COL_WIDTH) << left << CYAN_COLOR + "Length (M/F)" + RESET_COLOR << "|\n";
        cout << "|" << setw(COL_WIDTH) << left << CYAN_COLOR + "7" + RESET_COLOR
             << "|" << setw(COL_WIDTH) << left << CYAN_COLOR + "View History" + RESET_COLOR << "|\n";
        cout << "|" << setw(COL_WIDTH) << left << CYAN_COLOR + "8" + RESET_COLOR
             << "|" << setw(COL_WIDTH) << left << CYAN_COLOR + "Quit" + RESET_COLOR << "|\n";
        cout << "+" << string(COL_WIDTH, '-') << "+" << string(COL_WIDTH, '-') << "+\n";
    }

    void displayHistory() const {
        const int COL_WIDTH = 15;
        if (history.empty()) {
            cout << YELLOW_COLOR << "No history available." << RESET_COLOR << endl;
            return;
        }
        cout << "\n+" << string(COL_WIDTH, '-') << "+" << string(COL_WIDTH, '-') << "+" << string(COL_WIDTH, '-') << "+\n";
        cout << "|" << setw(COL_WIDTH) << left << BLUE_COLOR + "Type" + RESET_COLOR
             << "|" << setw(COL_WIDTH) << left << BLUE_COLOR + "Input" + RESET_COLOR
             << "|" << setw(COL_WIDTH) << left << BLUE_COLOR + "Result" + RESET_COLOR << "|\n";
        cout << "+" << string(COL_WIDTH, '-') << "+" << string(COL_WIDTH, '-') << "+" << string(COL_WIDTH, '-') << "+\n";
        for (const auto& entry : history) {
            cout << "|" << setw(COL_WIDTH) << left << entry.type
                 << "|" << setw(COL_WIDTH) << left << entry.input
                 << "|" << setw(COL_WIDTH) << left << GREEN_COLOR + entry.result + RESET_COLOR << "|\n";
        }
        cout << "+" << string(COL_WIDTH, '-') << "+" << string(COL_WIDTH, '-') << "+" << string(COL_WIDTH, '-') << "+\n";
    }

public:
    void run() {
        showWelcome();

        while (true) {
            displayMenu();
            int choice = static_cast<int>(getDoubleInput("Enter choice (1-8): "));
            clearInputBuffer();

            try {
                switch (choice) {
                    case 1: {
                        double num1 = getDoubleInput("Enter first number: ");
                        char op = getCharInput("Enter operation (+, -, *, /, ^, S(sin), C(cos), T(tan)): ");
                        if (op != 'S' && op != 'C' && op != 'T') {
                            double num2 = getDoubleInput("Enter second number: ");
                            Calculator calc(num1, op, num2);
                            calc.displayResult();
                            ostringstream oss;
                            oss << fixed << setprecision(2) << num1 << " " << op << " " << num2;
                            string inputStr = oss.str();
                            oss.str(""); oss << fixed << setprecision(2) << calc.calculate();
                            history.emplace_back("Calculator", inputStr, oss.str());
                        } else {
                            Calculator calc(num1, op);
                            calc.displayResult();
                            ostringstream oss;
                            oss << fixed << setprecision(2) << num1 << " " << op;
                            string inputStr = oss.str();
                            oss.str(""); oss << fixed << setprecision(2) << calc.calculate();
                            history.emplace_back("Calculator", inputStr, oss.str());
                        }
                        break;
                    }
                    case 2: {
                        double temp = getDoubleInput("Enter temperature: ");
                        char from = getCharInput("Enter from unit (C or F): ");
                        char to = getCharInput("Enter to unit (C or F): ");
                        TemperatureConverter tempConv(temp, from, to);
                        ostringstream oss;
                        oss << fixed << setprecision(2) << temp;
                        string inputStr = oss.str();
                        tempConv.displayResult(inputStr, string(1, toupper(from)) + " to " + string(1, toupper(to)));
                        oss.str(""); oss << fixed << setprecision(2) << tempConv.convert();
                        history.emplace_back(tempConv.getType(), inputStr + " " + string(1, toupper(from)) + " to " + string(1, toupper(to)), oss.str());
                        break;
                    }
                    case 3: {
                        string number = getStringInput("Enter number: ");
                        char from = getCharInput("Enter from base (B(binary), D(decimal), O(octal), H(hex)): ");
                        char to = getCharInput("Enter to base (B, D, O, H): ");
                        NumberBaseConverter baseConv(number, from, to);
                        baseConv.displayResult(number, string(1, toupper(from)) + " to " + string(1, toupper(to)));
                        history.emplace_back(baseConv.getType(), number + " " + string(1, toupper(from)) + " to " + string(1, toupper(to)), baseConv.getResultString());
                        break;
                    }
                    case 4: {
                        double num = getDoubleInput("Enter number: ");
                        char type = getCharInput("Enter log type (L=log10, N=ln, B=log2): ");
                        LogarithmicCalculator logCalc(num, type);
                        ostringstream oss;
                        oss << fixed << setprecision(2) << num;
                        string inputStr = oss.str();
                        string logStr = (type == 'L' ? "log10" : type == 'N' ? "ln" : "log2");
                        logCalc.displayResult(inputStr, logStr);
                        oss.str(""); oss << fixed << setprecision(2) << logCalc.convert();
                        history.emplace_back(logCalc.getType(), logStr + "(" + inputStr + ")", oss.str());
                        break;
                    }
                    case 5: {
                        double amount = getDoubleInput("Enter amount: ");
                        char from = getCharInput("Enter from currency (I, U, E, or G): ");
                        char to = getCharInput("Enter to currency (I, U, E, or G): ");
                        CurrencyConverter currConv(amount, from, to);
                        ostringstream oss;
                        oss << fixed << setprecision(2) << amount;
                        string inputStr = oss.str();
                        currConv.displayResult(inputStr, string(1, toupper(from)) + " to " + string(1, toupper(to)));
                            oss.str(""); oss << fixed << setprecision(2) << currConv.convert();
                        history.emplace_back(currConv.getType(), inputStr + " " + string(1, toupper(from)) + " to " + string(1, toupper(to)), oss.str());
                        break;
                    }
                    case 6: {
                        double length = getDoubleInput("Enter length: ");
                        char from = getCharInput("Enter from unit (M or F): ");
                        char to = getCharInput("Enter to unit (M or F): ");
                        LengthConverter lenConv(length, from, to);
                        ostringstream oss;
                        oss << fixed << setprecision(2) << length;
                        string inputStr = oss.str();
                        lenConv.displayResult(inputStr, string(1, toupper(from)) + " to " + string(1, toupper(to)));
                        oss.str(""); oss << fixed << setprecision(2) << lenConv.convert();
                        history.emplace_back(lenConv.getType(), inputStr + " " + string(1, toupper(from)) + " to " + string(1, toupper(to)), oss.str());
                        break;
                    }
                    case 7:
                        displayHistory();
                        break;
                    case 8: {
                        const int COL_WIDTH = 40;
                        cout << "+" << string(COL_WIDTH, '-') << "+\n";
                        cout << "|" << setw(COL_WIDTH) << left << CYAN_COLOR + "Thank you for using Professional Converter!" + RESET_COLOR << "|\n";
                        cout << "+" << string(COL_WIDTH, '-') << "+\n";
                        return;
                    }
                    default:
                        const int COL_WIDTH = 40;
                        cout << "+" << string(COL_WIDTH, '-') << "+\n";
                        cout << "|" << setw(COL_WIDTH) << left << RED_COLOR + "Invalid choice. Please select 1-8." + RESET_COLOR << "|\n";
                        cout << "+" << string(COL_WIDTH, '-') << "+\n";
                }
            } catch (const runtime_error& e) {
                const int COL_WIDTH = 45;
                cout << "+" << string(COL_WIDTH, '-') << "+\n";
                cout << "|" << setw(COL_WIDTH) << left << RED_COLOR + string("Error: ") + e.what() + RESET_COLOR << "|\n";
                cout << "+" << string(COL_WIDTH, '-') << "+\n";
            }
        }
    }
};

int main() {
    Program program;
    program.run();
    return 0;
}
