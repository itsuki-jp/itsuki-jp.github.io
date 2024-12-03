---
title: PythonでつくるPython
date: 2024-11-30T17:02:25+09:00
draft: true
summary: アドカレ4日目
---
どこかの、アドベントカレンダー4日目の記事です！！！

# はじめに・経緯
[ラムダノート](https://www.lambdanote.com/)を知っていますか？私も先輩から聞いて先日初めて知りましたが、技術書の出版社です。ここのいいところは、PDF版の値段＋数百円で紙媒体の本もついてくるところだと思います。PDFも紙媒体もお得に買える！ということで、[【RubyでつくるRuby ゼロから学びなおすプログラミング言語入門】](https://www.lambdanote.com/products/ruby-ruby)を買いました。

![alt text](/images/2024-12-04_runPythonOnPython/image.png)

この本の著者の方の【[『RubyでつくるRuby』の読み方（私論）](https://golden-lucky.hatenablog.com/entry/2023/01/10/131903)】に以下のように書いてありました。
> 『RubyでつくるRuby』を読みながら『PythonでつくるPython』とか『JavaScriptでつくるJavaScript』を自分でやってみることです。これには次の2つの意味でものすごい教育効果があると思います。
> - それぞれの言語のソースコードの文字列を読み込んで抽象構文木に変換するパーザを自分で書くことになる
> - それぞれの言語の挙動を、とくにRubyとの対比で、いま以上によく知れる

なので、この記事では【PythonでつくるPython】をしていこうと思います！！！

# PythonでPythonを動かそう
## コード
```py
# interp.py
import sys
from minPython import minPython

if len(sys.argv) < 2:
    print("Usage: python main.py <file_path>")
    sys.exit(1)

file_path = sys.argv[1]
min_python = minPython()

ast_tree = min_python.get_tree(file_path)
custom_tree = min_python.evaluate(ast_tree)


def evaluate(tree, global_var, local_var):
    if tree[0] == "lit":
        return tree[1]
    elif tree[0] == "+":
        return evaluate(tree[1], global_var, local_var) + evaluate(
            tree[2], global_var, local_var
        )
    elif tree[0] == "-":
        return evaluate(tree[1], global_var, local_var) - evaluate(
            tree[2], global_var, local_var
        )
    elif tree[0] == "*":
        return evaluate(tree[1], global_var, local_var) * evaluate(
            tree[2], global_var, local_var
        )
    elif tree[0] == "/":
        return evaluate(tree[1], global_var, local_var) / evaluate(
            tree[2], global_var, local_var
        )
    elif tree[0] == "==":
        return evaluate(tree[1], global_var, local_var) == evaluate(
            tree[2], global_var, local_var
        )
    elif tree[0] == "!=":
        return evaluate(tree[1], global_var, local_var) != evaluate(
            tree[2], global_var, local_var
        )
    elif tree[0] == "<":
        return evaluate(tree[1], global_var, local_var) < evaluate(
            tree[2], global_var, local_var
        )
    elif tree[0] == "<=":
        return evaluate(tree[1], global_var, local_var) <= evaluate(
            tree[2], global_var, local_var
        )
    elif tree[0] == ">":
        return evaluate(tree[1], global_var, local_var) > evaluate(
            tree[2], global_var, local_var
        )
    elif tree[0] == ">=":
        return evaluate(tree[1], global_var, local_var) >= evaluate(
            tree[2], global_var, local_var
        )
    elif tree[0] == "stmts":
        for node in tree[1:]:
            evaluate(node, global_var, local_var)
    elif tree[0] == "var_assign":
        local_var[tree[1]] = evaluate(tree[2], global_var, local_var)
    elif tree[0] == "subscript_assign":
        obj = evaluate(tree[1], global_var, local_var)
        index = evaluate(tree[2], global_var, local_var)
        value = evaluate(tree[3], global_var, local_var)
        obj[index] = value
        return obj
    elif tree[0] == "var_ref":
        return local_var[tree[1]]
    elif tree[0] == "if":
        if evaluate(tree[1], global_var, local_var):
            return evaluate(tree[2], global_var, local_var)
        else:
            return evaluate(tree[3], global_var, local_var)
    elif tree[0] == "func_call":
        try:
            func_name = tree[1]
            args = [evaluate(arg, global_var, local_var) for arg in tree[2:]]

            if func_name in global_var:
                new_lcoal_var = {}
                for i, arg in enumerate(args):
                    new_lcoal_var[global_var[func_name][1][i]] = arg
                return evaluate(global_var[func_name][2], global_var, new_lcoal_var)
            build_in_func = globals().get(func_name)
            if build_in_func is None:
                if isinstance(__builtins__, dict):
                    build_in_func = __builtins__.get(func_name)
                else:
                    build_in_func = getattr(__builtins__, func_name, None)
            if build_in_func:
                return build_in_func(*args)
            else:
                raise ValueError(f"Unknown function: {func_name}")
        except Exception as e:
            print(f"Error: {e}")
    elif tree[0] == "method_call":
        obj = evaluate(tree[1], global_var, local_var)
        method_name = tree[2]
        args = [evaluate(arg, global_var, local_var) for arg in tree[3:]]
        build_in_func = getattr(obj, method_name, None)
        if build_in_func:
            return build_in_func(*args)
    elif tree[0] == "func_def":
        global_var[tree[1]] = ["user_func", tree[2], tree[3]]
    elif tree[0] == "list_new":
        return [evaluate(elt, global_var, local_var) for elt in tree[1:]]
    elif tree[0] == "dict_new":
        dict_new = {}
        for i in range(1, len(tree), 2):
            dict_new[evaluate(tree[i], global_var, local_var)] = evaluate(
                tree[i + 1], global_var, local_var
            )
        return dict_new
    elif tree[0] == "dictlist_ref":
        return evaluate(tree[1], global_var, local_var)[
            evaluate(tree[2], global_var, local_var)
        ]
    else:
        raise ValueError(f"Unknown node: {tree}")


evaluate(custom_tree, {}, {})
```

## 解説
コードは以下のように、`tree[0]`が`+`であれば、再帰的に値を求めて、足し算をするようにしています。わかりやすくていいですね！

```py
elif tree[0] == "+":
    return evaluate(tree[1], global_var, local_var) + 
    evaluate(tree[2], global_var, local_var)
```

実行手順は`hogehoge.py` を用意して、`python interp.py hogehoge.py` を実行すれば動きます！簡単ですね！！！

まあ実際はそんなことはなく、最初のほうの`from minPython import minPython`がほぼすべての仕事をしてくれてます。(`interp.py`はきれいに成形された値を再帰的に求めてるだけです...)

### AST(抽象構文木)
`minPython`の話に移る前に、少し前の話をします。

PythonにはAST(抽象構文木)という、コードをいい感じに木で表現してくれるモジュールがあるのですが、そこそこ難解な形で投げ返してくれます...

以下のの二つは、Pythonの通常の四則演算のコードと、それをASTが投げ返してくる出力です。

```py
# 普通のPythonの四則演算
1 + 2 * 3 / (4 + 2)
1 / 2 / 3
```

```py
# ASTによる出力
 ('Module(
   body=[
       Expr(
           value=BinOp(
               left=Constant(value=1),
               op=Add(),
               right=BinOp(
                   left=BinOp(
                       left=Constant(value=2),
                       op=Mult(),
                       right=Constant(value=3)),
                   op=Div(),
                   right=BinOp(
                       left=Constant(value=4),
                       op=Add(),
                       right=Constant(value=2))))),
       Expr(
           value=BinOp(
               left=BinOp(
                   left=Constant(value=1),
                   op=Div(),
                   right=Constant(value=2)),
               op=Div(),
               right=Constant(value=3)))],
   type_ignores=[])')
```

見るだけでも、いやな気持になってきますね...。これをいい感じにするのが`minPython`です

### minPython?

`minPython`を使うと、先ほどの四則演算コードを以下のいい感じの形式に直してくれます。見やすくて気分が良いですね！

```py
[
    "stmts",
    [
        "+",
        ["lit", 1],
        ["/", ["*", ["lit", 2], ["lit", 3]], ["+", ["lit", 4], ["lit", 2]]],
    ],
    ["/", ["/", ["lit", 1], ["lit", 2]], ["lit", 3]],
]
```

`minPython`はこの世の誰かが既に書いてくれていそうですが、今回はせっかくなので書きました。

やってることは、あまり`interp.py`と変わらないものの、astのインスタンスの種類によって場合分けするのがめんどくさかったですね。

```py
# minPython.py
import ast
from icecream import ic


class minPython:
    def __init__(self):
        self.map_binOp = {"Add": "+", "Sub": "-", "Mult": "*", "Div": "/"}
        self.map_compare = {
            "Eq": "==",
            "NotEq": "!=",
            "Lt": "<",
            "LtE": "<=",
            "Gt": ">",
            "GtE": ">=",
        }

    def get_tree(self, path):
        """指定されたファイルをASTに変換する"""
        with open(path, "r") as file:
            tree = ast.parse(file.read())
            ic(ast.dump(tree, indent=4))
            return tree

    def evaluate(self, tree):
        """ASTを評価してカスタム表現に変換する"""
        if isinstance(tree, ast.Module):
            res = ["stmts"]
            for node in tree.body:
                res.append(self.evaluate(node))
            return res
        elif isinstance(tree, ast.Expr):
            return self.evaluate(tree.value)
        elif isinstance(tree, ast.BinOp):
            return [
                self.map_binOp[type(tree.op).__name__],
                self.evaluate(tree.left),
                self.evaluate(tree.right),
            ]
        elif isinstance(tree, ast.Constant):
            return ["lit", tree.value]
        elif isinstance(tree, ast.Assign):
            if isinstance(tree.targets[0], ast.Subscript):
                target = self.evaluate(tree.targets[0].value)
                index = self.evaluate(tree.targets[0].slice)
                value = self.evaluate(tree.value)
                return ["subscript_assign", target, index, value]
            else:
                return ["var_assign", tree.targets[0].id, self.evaluate(tree.value)]
        elif isinstance(tree, ast.Name):
            return ["var_ref", tree.id]
        elif isinstance(tree, ast.If):
            return [
                "if",
                self.evaluate(tree.test),
                self.evaluate(tree.body[0]),
                self.evaluate(tree.orelse[0]),
            ]
        elif isinstance(tree, ast.Compare):
            return [
                self.map_compare[type(tree.ops[0]).__name__],
                self.evaluate(tree.left),
                self.evaluate(tree.comparators[0]),
            ]
        # Todo: While をサポートする
        elif isinstance(tree, ast.Call):
            if isinstance(tree.func, ast.Name):
                return [
                    "func_call",
                    tree.func.id,
                    *[self.evaluate(arg) for arg in tree.args],
                ]
            elif isinstance(tree.func, ast.Attribute):
                return [
                    "method_call",
                    self.evaluate(tree.func.value),
                    tree.func.attr,
                    *[self.evaluate(arg) for arg in tree.args],
                ]
        elif isinstance(tree, ast.FunctionDef):
            return [
                "func_def",
                tree.name,
                [arg.arg for arg in tree.args.args],
                *[self.evaluate(node) for node in tree.body],
            ]
        elif isinstance(tree, ast.Return):
            return self.evaluate(tree.value)
        elif isinstance(tree, ast.List):
            return ["list_new", *[self.evaluate(elt) for elt in tree.elts]]
        elif isinstance(tree, ast.Dict):
            result = ["dict_new"]
            for k, v in zip(tree.keys, tree.values):
                result.append(self.evaluate(k))
                result.append(self.evaluate(v))
            return result
        elif isinstance(tree, ast.Subscript):
            return [
                "dictlist_ref",
                self.evaluate(tree.value),
                self.evaluate(tree.slice),
            ]
        else:
            return None
```

下の`assign`を見てもらえるとわかりやすいのですが、代入と一言で言っても、`a = 1`と`a[0] = 1`のような書き方があり、そこらへんをある程度網羅するのもめんどくさかったですね。
```py
elif isinstance(tree, ast.Assign):
    if isinstance(tree.targets[0], ast.Subscript):
        target = self.evaluate(tree.targets[0].value)
        index = self.evaluate(tree.targets[0].slice)
        value = self.evaluate(tree.value)
        return ["subscript_assign", target, index, value]
    else:
        return ["var_assign", tree.targets[0].id, self.evaluate(tree.value)]
```

# 最後に
実は（？）記事のタイトルは若干誇張表現です...。サポートしてるPythonの構文は以下の通りで、サポートしてないのも多くあります
- 四則演算
- 数値・文字の代入
- リスト・辞書の諸々
  - 代入(`hoge = [1,2,3]`)
  - append, remove系統
  - 特定の場所に代入(`hoge[1] = 2`)
- if 文
- for 文
- 自作関数の定義・呼び出し（上手くいかない場合もありそう）
- 組み込み関数の呼び出し（上手くいかない場合もありそう）

Whileはforでいいか～と思い、またの機会・気が向いたときに実装しようと思います

## TDD
あと、今回は`minPython`を作るときは、TDD(テスト駆動開発)をしてみました。出力がこうなってほしい！が定まっているので、先にテストを書いておいて、あとはそれに合うように実装するだけだったので、楽でしたね。GitHub Copilot 君も、テストを見てサポートしてくれたので、非常に楽でした。

```py
test_minPython.py
from minPython import minPython
from icecream import ic

min_python = minPython()


def test_calc():
    calc_tree = min_python.get_tree("test/calc.py")
    res = min_python.evaluate(calc_tree)
    exp = [
        "stmts",
        [
            "+",
            ["lit", 1],
            ["/", ["*", ["lit", 2], ["lit", 3]], ["+", ["lit", 4], ["lit", 2]]],
        ],
        ["/", ["/", ["lit", 1], ["lit", 2]], ["lit", 3]],
    ]
    ic(res)
    ic(exp)
    assert res == exp


def test_assign():
    assign_tree = min_python.get_tree("test/assign.py")
    res = min_python.evaluate(assign_tree)
    exp = ["stmts", ["var_assign", "x", ["lit", 1]], ["var_assign", "y", ["lit", 10]]]
    ic(res)
    ic(exp)
    assert res == exp


def test_assign_and_calc():
    assign_tree = min_python.get_tree("test/assign_and_calc.py")
    res = min_python.evaluate(assign_tree)
    exp = [
        "stmts",
        ["var_assign", "x", ["lit", 1]],
        ["var_assign", "y", ["lit", 10]],
        [
            "var_assign",
            "z",
            ["+", ["+", ["lit", 10], ["var_ref", "x"]], ["var_ref", "y"]],
        ],
    ]
    ic(res)
    ic(exp)
    assert res == exp


def test_condition():
    condition_tree = min_python.get_tree("test/condition.py")
    res = min_python.evaluate(condition_tree)
    exp = [
        "stmts",
        ["var_assign", "x", ["lit", 10]],
        ["var_assign", "res", ["lit", ""]],
        [
            "if",
            [">", ["var_ref", "x"], ["lit", 10]],
            ["var_assign", "res", ["lit", "big"]],
            ["var_assign", "res", ["lit", "small"]],
        ],
    ]
    ic(res)
    ic(exp)
    assert res == exp


def test_condition2():
    condition_tree = min_python.get_tree("test/condition2.py")
    res = min_python.evaluate(condition_tree)
    exp = [
        "stmts",
        ["var_assign", "a", ["lit", 1]],
        ["var_assign", "b", ["lit", 2]],
        ["var_assign", "x", ["==", ["var_ref", "a"], ["var_ref", "b"]]],
        ["var_assign", "y", [">", ["var_ref", "a"], ["var_ref", "b"]]],
        ["var_assign", "z", ["<", ["var_ref", "a"], ["var_ref", "b"]]],
        ["var_assign", "o", ["<=", ["var_ref", "a"], ["var_ref", "b"]]],
        ["var_assign", "p", [">=", ["var_ref", "a"], ["var_ref", "b"]]],
    ]
    ic(res)
    ic(exp)
    assert res == exp


def test_buildInFunction():
    buildInFunction_tree = min_python.get_tree("test/buildInFunction.py")
    res = min_python.evaluate(buildInFunction_tree)
    exp = [
        "stmts",
        ["func_call", "print", ["lit", "hell world"]],
        [
            "func_call",
            "print",
            ["func_call", "max", ["lit", 1], ["lit", 2], ["lit", 3], ["lit", 4]],
        ],
    ]
    ic(res)
    ic(exp)
    assert res == exp


def test_originalFunction():
    originalFunction_tree = min_python.get_tree("test/originalFunction.py")
    res = min_python.evaluate(originalFunction_tree)
    exp = [
        "stmts",
        [
            "func_def",
            "max_of_two",
            ["a", "b"],
            [
                "if",
                [">", ["var_ref", "a"], ["var_ref", "b"]],
                ["var_ref", "a"],
                ["var_ref", "b"],
            ],
        ],
        ["func_call", "print", ["func_call", "max_of_two", ["lit", 5], ["lit", 10]]],
    ]
    ic(res)
    ic(exp)
    assert res == exp


def test_list_dict():
    list_dict_tree = min_python.get_tree("test/list_dict.py")
    res = min_python.evaluate(list_dict_tree)
    exp = [
        "stmts",
        ["var_assign", "a", ["list_new", ["lit", 1], ["lit", 2], ["lit", 3]]],
        ["var_assign", "b", ["dictlist_ref", ["var_ref", "a"], ["lit", 1]]],
        ["func_call", "print", ["var_ref", "b"]],
        [
            "var_assign",
            "c",
            ["dict_new", ["lit", "a"], ["lit", 1], ["lit", "b"], ["lit", 2]],
        ],
        ["var_assign", "d", ["dictlist_ref", ["var_ref", "c"], ["lit", "a"]]],
        ["func_call", "print", ["var_ref", "d"]],
    ]
    ic(res)
    ic(exp)
    assert res == exp


def test_all():
    list_dict_tree = min_python.get_tree("test/all_test.py")
    res = min_python.evaluate(list_dict_tree)
    exp = [
        "stmts",
        ["var_assign", "var_int", ["lit", 100]],
        ["var_assign", "var_str", ["lit", "Hello, World!"]],
        ["var_assign", "var_list", ["list_new", ["lit", 1], ["lit", 2], ["lit", 3]]],
        ["method_call", ["var_ref", "var_list"], "append", ["lit", 4]],
        [
            "var_assign",
            "var_dict",
            ["dict_new", ["lit", "a"], ["lit", 1], ["lit", "b"], ["lit", 2]],
        ],
        ["var_assign", "var_bool", ["lit", True]],
        ["func_call", "print", ["var_ref", "var_int"], ["var_ref", "var_int"]],
        ["func_call", "print", ["var_ref", "var_str"]],
        ["func_call", "print", ["var_ref", "var_list"]],
        ["func_call", "print", ["var_ref", "var_dict"]],
        ["func_call", "print", ["var_ref", "var_bool"]],
        [
            "func_def",
            "get_max",
            ["a", "b"],
            [
                "if",
                [">", ["var_ref", "a"], ["var_ref", "b"]],
                ["var_ref", "a"],
                ["var_ref", "b"],
            ],
        ],
        [
            "func_call",
            "print",
            ["func_call", "get_max", ["var_ref", "var_int"], ["lit", 200]],
        ],
    ]
    ic(res)
    ic(exp)
    assert res == exp
```

```py
# all_test.py
# 四則演算、比較演算子、 代入、変数参照、If、関数呼び出し、メソッド実行、関数定義、リスト、辞書、リスト/辞書参照をサポート


var_int = 100
var_str = "Hello, World!"
var_list = [1, 2, 3]
var_list.append(4)
var_dict = {"a": 1, "b": 2}
var_bool = True

print(var_int, var_int)
print(var_str)
print(var_list)
print(var_dict)
print(var_bool)


def get_max(a, b):
    if a > b:
        return a
    else:
        return b


print(get_max(var_int, 200))
```

```py
# 一番最後に、interp.pの諸々が上手くいってるか試すためのコード

# 1. リテラルと変数の割り当て
var_int = 10
print("var_int:", var_int)
var_float = 5.5
print("var_float:", var_float)
var_str = "Hello"
print("var_str:", var_str)
var_list = [1, 2, 3]
print("var_list:", var_list)
var_dict = {"key1": "value1", "key2": "value2"}
print("var_dict:", var_dict)

# 2. 算術演算
var_sum = var_int + 20
print("var_sum:", var_sum)
var_product = var_int * 2
print("var_product:", var_product)
var_division = var_float / 2.0
print("var_division:", var_division)
var_difference = var_int - 5
print("var_difference:", var_difference)

# 3. 比較演算
is_equal = var_int == 10
print("is_equal:", is_equal)
is_not_equal = var_float != 3.0
print("is_not_equal:", is_not_equal)
is_less_than = var_int < 15
print("is_less_than:", is_less_than)
is_less_equal = var_int <= 10
print("is_less_equal:", is_less_equal)
is_greater_than = var_int > 5
print("is_greater_than:", is_greater_than)
is_greater_equal = var_int >= 10
print("is_greater_equal:", is_greater_equal)

# 4. 条件分岐
if var_int > 5:
    var_condition = "Greater"
else:
    var_condition = "Lesser"
print("var_condition:", var_condition)


# 5. 関数定義と呼び出し
def add(a, b):
    return a + b


result_add = add(var_int, var_float)
print("result_add:", result_add)

# 6. リスト操作
var_list.append(4)  # メソッド呼び出し
var_list[1] = 5  # インデックスで更新
print("var_list after append:", var_list)
list_value = var_list[2]  # インデックスで参照
print("list_value:", list_value)

# 7. 辞書操作
var_dict["key3"] = "value3"
print("var_dict after update:", var_dict)
dict_value = var_dict["key1"]
print("dict_value:", dict_value)

# 8. ネストしたリストと辞書操作
nested = {"list": [1, {"key": "value"}]}
nested_value = nested["list"][1]["key"]
print("nested_value:", nested_value)

# 9. 組み込み関数の呼び出し
print("print example:", var_int, var_float)  # 組み込み関数 print
```