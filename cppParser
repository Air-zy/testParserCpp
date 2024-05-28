#include <iostream>
#include <windows.h>
#include <string>
#include <fstream>

#include <sstream>
#include <vector>
#include <cctype>

#include <algorithm> // For std::find

void setTxtColor(int colorValue);
void removeQuotes(std::string& str);
void removeWhitespace(std::string& str);
std::string addSpaces(int amt);
int getNumberDigitCount(int number) { return std::to_string(number).length(); }

std::string askForFilePath() {
    std::string filePath = "";
    std::getline(std::cin, filePath);
    removeWhitespace(filePath);
    removeQuotes(filePath);
    return filePath;
};

// constant global variables lookup
struct TOKENIDS {
  int number = 1;
  int word = 2;
  int symbol = 3;
  int includes = 4;
  int varType = 5;
  int varName = 6;
  int returnTypes = 7; // void
  int funcName = 8;
  int doubleEquals = 9; // ==
  int doubleSemiCols = 10; // ::
  int lineComment = 11; // //
  int isString = 12; // "hello world"
  int isChar = 13; // '\n'
  int logicAnd = 14; // &&
  int logicOr = 15; // ||
  int equalOperator = 16; // =
  int lessThanOpert = 17;  // <
  int moreThanOpert = 18;  // >
  int EquallessThanOpert = 19;  // <=
  int EqualmoreThanOpert = 20;  // >=
  int returnKeyWord = 21; // return;
  int ifKeyWord = 22; // if (
  int forKeyWord = 23; // for (
  int whileKeyWord = 24; // while (
  int doKeyWord = 25; // do {
  int functionCall = 26; // getSum(
  int elseKeyWord = 27;   // else
  int breakKeyWord = 28; // break;
  int continueKeyWord = 29; // continue;
  int variableModifiers = 30; // static, const, constexpr, inline, friend // unsigned
  int structure = 31; // struct
  int classObject = 32; // class
  int headerVariableName = 33; // func (int number
  int multiLineComment = 34; /* these */
  int otherReservedWords = 35; // using namespace
  int dotOperator = 36; // .
  int insertionOp = 37; // <<
  int extractionOp = 38;  // >>
};

constexpr TOKENIDS IDS;
int getTokenColor(int tokenId) {
  switch (tokenId) {
    case IDS.multiLineComment:
    case IDS.lineComment:
      return 8;  // grey
      break;
    case IDS.includes:
      return 10;  // green
      break;
    case IDS.number:
      return 11;  // lightest blue
      break;
    case IDS.varType:
    case IDS.returnTypes:
      return 9;  // lighter blue
      break;
    case IDS.structure:
    case IDS.classObject:
    case IDS.varName:
    case IDS.variableModifiers:
    case IDS.funcName:
      return 3;  // blue
      break;
    case IDS.isString:
      return 10;  // light green
      break;
    case IDS.isChar:
      return 6;  // dark yellow
      break;
    case IDS.doKeyWord:
    case IDS.whileKeyWord:
    case IDS.forKeyWord:
    case IDS.ifKeyWord:
    case IDS.returnKeyWord:
    case IDS.breakKeyWord:
    case IDS.continueKeyWord:
    case IDS.elseKeyWord:
      return 13;  // purple
      break;
    case IDS.functionCall:
      return 14; // light yellow
      break;
    case IDS.otherReservedWords:
      return 12; // brighr red
      break;
    default:
      return 7;  // almost bright white
      break;
  }
}

struct Token {
    std::string text = "";
    unsigned short id = 0;
    unsigned int line = 0;
    unsigned int nest = 0;  // the like tabbing
    unsigned int nestId = 0;  // if of nest, diff for all nests
    bool inHeader = false; // for function headers and stuff like if (blah
};

bool isIdentifierChar(char c) {
    return std::isalnum(c) || c == '_';
}

bool isCppSymbol(char c) {
    static const std::string symbols = "\"\'!%^&*()-+=|[]{};:,?/.";
    return symbols.find(c) != std::string::npos;
}

std::vector<Token> parseFile(std::string filePath) {
    std::ifstream file(filePath);
    if (!file.is_open()) {
        std::cout << "[ERROR, file is open] " << filePath << '\n';
        exit(1);
    }
    //
    std::vector<Token> tokens;
    std::string line;
    int lineCount = 0;
    int currentTab = 0;
    int currentNestId = 0;

    bool inMultiLineComment = false;
    while (std::getline(file, line)) {
        std::string preprocessedLine;
        size_t lineSize = line.size();
        for (size_t i = 0; i < lineSize; ++i) {
            if (isCppSymbol(line[i])) {
                preprocessedLine += " ";
                preprocessedLine += line[i];
                preprocessedLine += " ";
            } else if (line[i] == '=') {
                preprocessedLine += " = ";
            } else {
                preprocessedLine += line[i];
            }
        }

        std::istringstream lineStream(preprocessedLine);
        std::string word;

        bool inSingleLineComment = false;
        bool inCharacterDeclare = false;
        int charSizeCounter = 0;

        bool inSingleLineString = false;
        bool inHeaders = false;
        while (lineStream >> word) {
            Token token;
            token.text = word;

            token.nestId = currentNestId;
            if (!inSingleLineComment && !inMultiLineComment && !inCharacterDeclare && !inSingleLineString) {
              if (std::isdigit(word[0])) {
                token.id = IDS.number;  // Number
              } else if (std::isalpha(word[0]) || word[0] == '_') {
                token.id = IDS.word;
              } else {
                token.id = IDS.symbol;  // Symbol
              }

              if (word == "int" || word == "double" || word == "short" ||
                  word == "long" || word == "bool" || word == "char" ||
                  word == "string") {
                token.id = IDS.varType;  // varb type
              }
              if (word == "void") {
                token.id = IDS.returnTypes;  // return type
              }

              if (word == "{") {
                currentTab += 1;
                ++currentNestId;
              } else if (word == "}") {
                currentTab -= 1;
              }
            }
            token.nest = currentTab;
            if (!tokens.empty()) {
                Token& previous = tokens.back();

                // string stuff
                if (previous.text == "\"") {
                    if (inSingleLineString == true){
                        inSingleLineString = false;
                    } else {
                        inSingleLineString = true;
                        previous.id = IDS.isString;
                    }
                }
                char lastChar = previous.text[previous.text.length() - 1];
                if (word == "\"" && inSingleLineString && lastChar != '\\') {
                    previous.text = previous.text + word;
                    inSingleLineString = false;
                    continue;
                }
                if (inSingleLineString) {
                    if (previous.text == "\"" || word == "\"") {
                        previous.text = previous.text + word;
                    } else {
                        previous.text = previous.text + ' ' + word;
                    }
                    continue;
                }
                 // char stuff
                if (previous.text == "'") {
                    if (inCharacterDeclare == true){
                        inCharacterDeclare = false;
                    } else {
                        inCharacterDeclare = true;
                        previous.id = IDS.isChar;
                    }
                }
                if (word == "'" && inCharacterDeclare) {
                    if (charSizeCounter == 0) {
                       previous.text = previous.text + ' ';
                    }
                    previous.text = previous.text + word;
                    inCharacterDeclare = false;
                    charSizeCounter = 0;
                    continue;
                }
                if (inCharacterDeclare) {
                    if (previous.text == "'" || word == "'") {
                        previous.text = previous.text + word;
                    } else {
                        previous.text = previous.text + ' ' + word;
                    }
                    ++charSizeCounter;
                    continue;
                }

                if (previous.id == IDS.varType || previous.id == IDS.returnTypes) {  // varb type || return type
                  if (inHeaders) {
                    token.id = IDS.headerVariableName;
                  } else {
                    token.id = IDS.varName;  // varb name
                  }
                  if (word == "*" || word == "&") { // pointer | reference
                    previous.text = previous.text + word;
                    continue;
                  }
                }

                if (previous.text == "return") {
                  previous.id = IDS.returnKeyWord;
                }
                if (previous.text == "break" && word == ";") {
                  previous.id = IDS.breakKeyWord;
                }
                if (previous.text == "continue" && word == ";") {
                  previous.id = IDS.continueKeyWord;
                }

                if (token.text == "(" && (previous.id == IDS.varName || previous.id == IDS.funcName)) {  // see if its a function // void func(
                  previous.id = IDS.funcName;
                  inHeaders = true;
                }
                if (token.text == ")" && inHeaders) {
                  inHeaders = false;
                }

                if (token.text == "(" && previous.id == IDS.word) {  // see if its a function call
                  previous.id = IDS.functionCall;
                }

                //line stuff
                if (previous.text == "{" || previous.text == ";") {
                  if (!inHeaders) {
                    ++lineCount;
                  }
                }
                if (previous.text == "}" && word != ";") {
                    ++lineCount;
                }
                if (previous.text == "#include" ) {
                    previous.text = previous.text + ' ' + word;
                    previous.id = IDS.includes;
                    ++lineCount;
                    continue;
                }
                if (
                    previous.id == IDS.includes
                    && (word == "." || token.id == IDS.symbol || token.id == IDS.word)
                    && previous.text.find('>') == std::string::npos
                    ){
                  previous.text = previous.text + word;
                  continue;
                }

                if (previous.text == "struct") {
                  previous.id = IDS.structure;
                }
                if (previous.text == "class") {
                  previous.id = IDS.classObject;
                }
                if (previous.text == "if" && word == "(") {
                  previous.id = IDS.ifKeyWord;
                }
                if (previous.text == "for" && word == "(") {
                  previous.id = IDS.forKeyWord;
                }
                if (previous.text == "while" && word == "(") {
                  previous.id = IDS.whileKeyWord;
                }
                if (previous.text == "do" && word == "{") {
                  previous.id = IDS.doKeyWord;
                }
                if (previous.text == "else" && previous.id == IDS.word) {
                  previous.id = IDS.elseKeyWord;
                }
                if ((
                    previous.text == "unsigned" ||
                    previous.text == "const" ||
                    previous.text == "constexpr" ||
                    previous.text == "static" ||
                    previous.text == "inline" ||
                    previous.text == "friend"
                    ) && previous.id == IDS.word) {
                  previous.id = IDS.variableModifiers;
                }
                if (previous.text == "using" && previous.id == IDS.word) {
                  previous.id = IDS.otherReservedWords;
                }
                if (previous.text == "namespace" && previous.id == IDS.word) {
                  previous.id = IDS.otherReservedWords;
                }

                //
                if (previous.text == "=" && word == "=") { // double equals
                    previous.text = "==";
                    previous.id = IDS.doubleEquals;
                    continue;
                }
                if (previous.text == "=" && word != "=") {
                    previous.id = IDS.equalOperator;
                }
                if (previous.text == "." && previous.id == IDS.symbol) {
                    previous.id = IDS.dotOperator;
                }
                if (previous.text == ":" && word == ":") { // double semi
                    previous.text = "::";
                    previous.id = IDS.doubleSemiCols;
                    continue;
                }
                if (previous.text == "/" && word == "/") { // single line comment
                    previous.text = "//";
                    previous.id = IDS.lineComment;
                    inSingleLineComment = true;
                    continue;
                }
                if (previous.text == "/" && word == "*") { // multi line comment START
                    previous.text = "/*";
                    previous.id = IDS.multiLineComment;
                    inMultiLineComment = true;
                    continue;
                }
                if (lastChar = '*' && word == "/" && inMultiLineComment) {  // multi line comment END
                    previous.text = previous.text + "/";
                    inMultiLineComment = false;
                    ++lineCount;
                    continue;
                }
                if (previous.text == "&" && word == "&") { // and
                    previous.text = "&&";
                    previous.id = IDS.logicAnd;
                    continue;
                }
                if (previous.text == "|" && word == "|") { // or
                    previous.text = "||";
                    previous.id = IDS.logicOr;
                    continue;
                }
                if (previous.text == ">" && word == "=") {
                    previous.text = ">=";
                    previous.id = IDS.EqualmoreThanOpert;
                    continue;
                }
                if (previous.text == "<" && word == "=") {
                    previous.text = "<=";
                    previous.id = IDS.EquallessThanOpert;
                    continue;
                }
                if (previous.text == ">" && word != ">") {
                  previous.id = IDS.moreThanOpert;
                }
                if (previous.text == "<" && word != "<") {
                  previous.id = IDS.lessThanOpert;
                }
                if (previous.text == ">>") {
                  previous.id = IDS.extractionOp;
                }
                if (previous.text == "<<") {
                  previous.id = IDS.insertionOp;
                }
                if (inSingleLineComment) {
                    previous.text = previous.text + ' ' + word;
                    continue;
                }
                if (inMultiLineComment) {
                  previous.text = previous.text + ' ' + word;
                  continue;
                }
            }

            std::cout << ".";
            token.line = lineCount;
            token.inHeader = inHeaders;
            tokens.push_back(token);
        }
        if (inSingleLineComment) {
            ++lineCount;
        }
    }

    file.close();
    return tokens;
}

void printTokens(std::vector<Token>&& tokens) {
  for (const Token& token : tokens) {

    for (int i = 0; i < token.nest; ++i) {
      std::cout << "    ";
    }

    std::cout << "[";
    setTxtColor(getTokenColor(token.id));
    std::cout << token.text;
    setTxtColor(15);
    std::cout << "] id: "
        << token.id
        << "\n";
  }
  std::cout << "\n\n";
  system("PAUSE");
}

void displayCode(std::vector<Token>&& tokens) {
    std::cout << "\n";
    int previouseLine = 0;
    const size_t totalTokensSize = tokens.size();
    for (size_t i = 0; i < totalTokensSize; ++i) {
        const Token& token = tokens[i];
        setTxtColor(getTokenColor(token.id));
        if (token.line != previouseLine) {
            std::cout << '\n';
            for (int i = 0; i < token.nest; ++i) {
                std::cout << "    ";
            }
        }
        std::cout << token.text;
        setTxtColor(15);
        previouseLine = token.line;
        if (token.id == IDS.number ||
            token.id == IDS.symbol ||
            token.id == IDS.isString ||
            token.id == IDS.isChar ||
            token.id == IDS.functionCall || 
            token.id == IDS.breakKeyWord ||
            token.id == IDS.continueKeyWord ||
            token.id == IDS.doubleSemiCols ||
            token.id == IDS.dotOperator
            ) {
          if (i + 1 < totalTokensSize) { // to make it so ){ this has space in ebtween
            const Token& nextToken = tokens[i + 1];
            if (nextToken.text != "{") {
              continue;
            }
          } else {
            continue;
          }
        }
        if (i + 1 < totalTokensSize) {
          const Token& nextToken = tokens[i + 1];
          if (nextToken.id == IDS.dotOperator) {
            continue;
          }
          if (nextToken.id == IDS.symbol && token.id == IDS.word) {
            continue;
          }
        }
        std::cout << ' ';
    }
    std::cout << "\n\n";
    system("PAUSE");
}

struct proccessedData {
  int lineCount = 0;
  int deepestNest = 0;
  int nestCount = 0;
  int ifStatementCount = 0;

  int globalVarCount = 0;
  int memberVarCount = 0;
  int headerVarCount = 0;

  int globalFuncCount = 0;
  int memberFuncCount = 0;

  int commentCount = 0;
  int multiLineComments = 0;
  int functionCalls = 0;
};

proccessedData processTokens(std::vector<Token>&& tokens) {
  proccessedData returnData;
  for (const Token& token : tokens) {
    if (token.line > returnData.lineCount) {
      returnData.lineCount = token.line;
    }
    if (token.nest > returnData.deepestNest) {
      returnData.deepestNest = token.nest;
    }
    if (token.nestId > returnData.nestCount) {
      returnData.nestCount = token.nestId;
    }
    if (token.id == IDS.ifKeyWord) {
      ++returnData.ifStatementCount;
    }
    if (token.id == IDS.varName) {
      if (token.nest == 0) {
        ++returnData.globalVarCount;
      } else {
        ++returnData.memberVarCount;
      }
    }
    if (token.id == IDS.headerVariableName) {
      ++returnData.headerVarCount;
    }
    if (token.id == IDS.lineComment) {
      ++returnData.commentCount;
    }
    if (token.id == IDS.multiLineComment) {
      ++returnData.multiLineComments;
    }
    if (token.id == IDS.functionCall && token.nest != 0) {
      ++returnData.functionCalls;
    }
    if (token.id == IDS.funcName) {
      if (token.text != "main") {
        if (token.nest == 0) {
          ++returnData.globalFuncCount;
        } else {
          ++returnData.memberFuncCount;
        }
      }
    }
  }
  return returnData;
}

int main() {
    std::cout << "\nenter ";
    setTxtColor(3);
    std::cout << "cpp ";
    setTxtColor(15);

    std::cout << "file_path to ";
    setTxtColor(3);
    std::cout << "read";
    setTxtColor(15);
    std::cout << ": ";

    //std::cin.ignore(1000, '\n');
    std::string filePath = askForFilePath();
    std::vector<Token> tokens = parseFile(filePath);
    proccessedData Data = processTokens(std::move(tokens));
    std::string input;

    int secondSpacing = 6;
    do {
      system("CLS");
      std::cout << " - " << filePath << " -\n"
                << "\n                                         #of if statements      " << Data.ifStatementCount << addSpaces(secondSpacing-getNumberDigitCount(Data.ifStatementCount)) << "| #of global functions: " << Data.globalFuncCount
                << "\n[v] find variable naming mistakes        deepest nest:          " << Data.deepestNest << addSpaces(secondSpacing-getNumberDigitCount(Data.deepestNest))           << "| #of member functions: " << Data.memberFuncCount
                << "\n[d] display code                         #of nests:             " << Data.nestCount << addSpaces(secondSpacing-getNumberDigitCount(Data.nestCount))               << "| #of function calls:   " << Data.functionCalls
                << "\n[t] display tokens                       #of local variables:   " << Data.memberVarCount << addSpaces(secondSpacing-getNumberDigitCount(Data.memberVarCount))     << "| "
                << "\n[q] quit                                 #of global variables:  " << Data.globalVarCount << addSpaces(secondSpacing-getNumberDigitCount(Data.globalVarCount))     << "| "
                << "\n                                         #of header variables:  " << Data.headerVarCount << addSpaces(secondSpacing-getNumberDigitCount(Data.headerVarCount))     << "|"
                << "\n                                         #of line/instructions: " << Data.lineCount << addSpaces(secondSpacing-getNumberDigitCount(Data.lineCount))               << "| commenting: " << Data.commentCount << " lines, " << Data.multiLineComments << " blocks"
                << "\ninput: ";
      std::cin >> input;
      if (input == "d") {
        displayCode(std::move(tokens));
      } else if (input == "t") {
        printTokens(std::move(tokens));
      }
    } while (input != "q");
    return 0;
}

void setTxtColor(int colorValue) {
    HANDLE hConsole;
    hConsole = GetStdHandle(STD_OUTPUT_HANDLE);
    SetConsoleTextAttribute(hConsole, colorValue);
}

// remove quotes
void removeQuotes(std::string& str) {
    if (!str.empty() && str.front() == '"') {
        str.erase(0, 1);
    }
    if (!str.empty() && str.back() == '"') {
        str.pop_back();
    }
};

void removeWhitespace(std::string& str) {
    // remove leading whitespace
    size_t start = 0;
    while (start < str.length() && std::isspace(str[start])) {
        start++;
    }
    str.erase(0, start);

    // remove trailing whitespace
    size_t strEnd = str.length();
    while (strEnd > 0 && std::isspace(str[strEnd - 1])) {
        strEnd--;
    }
    str.erase(strEnd);
}

// return a string of spaces of length amt given
std::string addSpaces(int amt) {  // pseudo set w so i get to control it more
  std::string spaces = "";
  for (int i = 0; i < amt; ++i) {
    spaces += ' ';
  }
  return spaces;
}
