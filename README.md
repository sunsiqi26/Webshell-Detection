# Webshell Detection

## PHP Webshell
PHP Webshell is essentially a piece of PHP code. If you use PHP's source code analysis directly, there will be a lot of noise, such as comment content, flower operations, etc. Converting the PHP Webshell source code into content that only executes statements will filter out this noise to some extent. Therefore, this project borrows VLD extension to convert PHP code into PHP opcode, and then uses bag of words and word frequency to extract key features for opcode data types, and finally uses GaussianNB for training.

## Project structure
```
.
├── black-list  #Folder for blackliste files
├── white-list-file-fold  #Folder for whiteliste files
├── __pycache__
│   └── utils.cpython-36.pyc
├── requirements.txt
├── save 
│   └── gnb.pkl
├── check.py
├── train.py
├── utils.py
├── utils.pyc
├── black_opcodes.txt
└── white_opcodes.txt
4 directories, 949 files
```

## Configuration environment
* Ubuntu 18.04
* Python 3.6
* PHP 5.6
* vld 0.14.0

## Dataset
- whitelist: [https://github.com/WordPress/WordPress](https://github.com/WordPress/WordPress)
- blacklist: [https://github.com/ysrc/webshell-sample](https://github.com/ysrc/webshell-sample)

## Data processing
- Extract opcode
Use Python subprocess module to perform system operations, get all its output, extract regular opcodes, and concatenate them with spaces. Iterate through all the PHP files in the target folder and generate opcode, and finally generate a list to write to black_opcodes.txt and white_opcodes.txt respectively.
```
def load_php_opcode(phpfilename):
try:
    output = subprocess.check_output(['php', '-dvld.active=1', '-dvld.execute=0', phpfilename], stderr=subprocess.STDOUT)
    output = str(output, encoding='utf-8')
    tokens = re.findall(r'\s(\b[A-Z_]+\b)\s', output)
    t = " ".join(tokens)
    return t
except:
    return " "
```
- Tagging
Label the PHP opcode in the white list with the tag [0], and label the PHP opcode in the black list with the tag [1]


## Function
- CountVectorizer
```
cv = CountVectorizer(ngram_range=(3, 3), decode_error="ignore", token_pattern=r'\b\w+\b')
X = cv.fit_transform(X).toarray()
```
Converts a collection of documents into a numeric matrix.
- TfidfTransformer
```
transformer = TfidfTransformer(smooth_idf=False)
X = transformer.fit_transform(X).toarray()
```
Normalize a numeric matrix to tf-idf
- train_test_split

Assign training and test sets
- GaussianNB
```
gnb = GaussianNB()
gnb.fit(x_train, y_train)
y_pred = gnb.predict(x_test)
```
Training with Naive Bayes Algorithm

## Result
> Accuracy :0.8936170212765957

## System flow chart
<img style="width:250px;height:500px" src="https://pic.downk.cc/item/5e75736a9d7d586a54892936.jpg" />

