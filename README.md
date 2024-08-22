# **Разработка транслятора языка ST на язык C**

## Поставнока задачи
Создание транслятора, который должен переводить ограниченное число выбранных грамматик языка ST, например, операторы ветвления, циклов, присваивания, на язык C. 
Программа транслятора должна быть написана с использованием лексического анализатора Flex и синтаксического анализатора Bison.

## Архитектура программы
### Устройство программы
Процесс выполнения программы можно разделить на две части:
1. Чтение и анализ текста на исходном языке.
2. Создание итогового транслированного кода.

Проект состоит из .l и .y файлов, а так же файла input.txt, в котором хранится исходный текст программы для транслирования на языке ST. 
Flex-парсер читает входной текст, передавая Bison токены, терминальные символы грамматики. 
Запустив Bison с ключом –d, получим .h-файл с перечислением всех терминалов, поэтому в .y-файле Bison объявим все возможные терминалы, а в .l-файле включим полученный заголовок.
Так flex-парсер проходит через входной текст, передавая обработанную информацию продвинутому Bison-парсеру.

### Лексемы и лексический файл

Программа Lex генерирует лексические анализаторы(lexer). Lexer принимает на входе поток символов, а когда встречает группу символов, совпадающих с некоторым шаблоном, то есть ключом, выполняет определенное действие.
На рисунке изображен Lex-файл. Разберем на примере показанного на рисунке кода что выполняет программа. Видим в строке 28 определение регулярного выражения [0-9]+. Это означает поиск последовательности из одного или более символов, имеющихся в группе 0123456789.
Его можно было бы записать сложнее – [0123456789]+. А, например, в строке 48 описывается словесная последовательность - [a-zA-Z][a-zA-Z0-9]*. Первая часть описывает совпадение в одном и только одном символе - это должен быть символ от 'a' до 'z', или от 'A' до 'Z'.
Другими словами, буквы латинского алфавита. После этой начальной буквы должно идти 0 или более символов - букв или цифр. Почему здесь используется звездочка? Символ '+' означает 1 или более совпадений, но слово ведь может состоять и из одного символа, и это совпадение уже описали ранее.
Поэтому вторая часть выражения может иметь 0 совпадений, поэтому пишем '*'. И так со всеми токенами: типами выражений, регулярными выражениями и так далее. 
Таким образом, мы имитируем логику многих языков программирования, требованием которых является то, что имя переменной обязательно начинается с буквы, но может содержать и цифры.

![image](https://github.com/user-attachments/assets/e0d50bd3-ad0b-46ce-b453-c51fd0bffb98)

Посредством команды flex flex.l мы компилируем наш .l файл и на выходе получаем другой файл lex.yy.c, который представляет из себя лексический анализатор и конечный детерминированный автомат.
Посредством команды bison -d bison.y компилируем файл bison.y и на выходе получаем два файла: bison.tab.c и bison.tab.h. Получился синтаксический анализатор Bison.
Это на самом деле функция на Си ( файл bison.tab.c), которая вызывается yyparse() из файла bison.y, то есть она запускает синтаксический анализатор, используя уже ранее указанные грамматические правила.
В этом файле bison.tab.c читаются все лексемы, которые указали в файле flex.l, выполняет действия и в конце концов завершает работу, когда встречает конец файла.
На рисунке представлен файл .y, в котором прописаны правила грамматики.

![image](https://github.com/user-attachments/assets/38fee0f2-1c83-4975-b248-483ec5928e5b)

yylex у нас определена в отдельном исходном файле, поэтому и выполняем компиляцию bison.y с помощью команды -d. Это нужно для того, чтоб он записал макроопределения лексем из файла .l в отдельный файл заголовка bison.tab.h. 
В нем эти лексемы прописаны и имеют нумерацию >257. В правиле Bison каждая лексема имеет значение. Значение группы, собираемой текущим правилом, хранится в $, значения остальных лексем — в $1, $2,…
Значение, хранящееся в $n есть не что иное, как значение переменной yylval в момент, когда лексер возвращает токен.

## Демонстрация работы
![image](https://github.com/user-attachments/assets/432bf7e1-1a1e-4296-ab9b-a92590833caf)