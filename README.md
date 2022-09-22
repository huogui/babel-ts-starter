# babel-ts-starter

# babel的编译流程 
  - parse 把源代码parse成AST
  - transform transform 阶段是对 parse 生成的 AST 的处理，会进行 AST 的遍历，遍历的过程中处理到不同的 AST 节点会调用注册的相应的 visitor 函数，visitor 函数里可以对 AST 节点进行增删改，返回新的 AST（可以指定是否继续遍历新生成的 AST）。这样遍历完一遍 AST 之后就完成了对代码的修改
  - generate generate 阶段会把 AST 打印成目标代码字符串，并且会生成 sourcemap。不同的 AST 对应的不同结构的字符串。比如 IfStatement 就可以打印成 if(test) {} 格式的代码。这样从 AST 根节点进行递归的字符串拼接，就可以生成目标代码的字符串。
# AST
  - 公共属性：
  - type : AST节点的类型
  - start、end、loc：start 和 end 代表该节点在源码中的开始和结束下标。而 loc 属性是一个对象，有 line 和 column 属性分别记录开始和结束的行列号。
  - leadingComments、innerComments、trailingComments： 表示开始的注释、中间的注释、结尾的注释，每个 AST 节点中都可能存在注释，而且可能在开始、中间、结束这三种位置，想拿到某个 AST 的注释就通过这三个属性。
  - extra：记录一些额外的信息，用于处理一些特殊情况。比如 StringLiteral 的 value 只是值的修改，而修改 extra.raw 则可以连同单双引号一起修改。
# babel 的 api
  - parse阶段有@babel/parse,功能是把源码转成AST
  - transform 阶段有 @babel/traverse，可以遍历 AST，并调用 visitor 函数修改 AST，修改 AST 自然涉及到 AST 的判断、创建、修改等，这时候就需要 @babel/types 了，当需要批量创建 AST 的时候可以使用 @babel/template 来简化 AST 创建逻辑。
  - generate 阶段会把 AST 打印为目标代码字符串，同时生成 sourcemap，需要 @babel/generator 包
  - 中途遇到错误想打印代码位置的时候，使用 @babel/code-frame 包
  - babel 的整体功能通过 @babel/core 提供，基于上面的包完成 babel 整体的编译流程，并应用 plugin 和 preset。
  - @babel/parse
  ``` ts
        function parse(input: string, options?: ParserOptions): File
        function parseExpression(input: string, options?: ParserOptions): Expression
   ```
  - @babel/traverse
        function traverse(parent,opts)
  - 遍历过程
        visitor 是指定对什么 AST 做什么处理的函数，babel 会在遍历到对应的 AST 时回调它们。而且可以指定刚开始遍历（enter）和遍历结束后（exit）两个阶段的回调函数，
        ``` js
                traverse(ast, {
                    FunctionDeclaration: {
                        enter(path, state) {}, // 进入节点时调用
                        exit(path, state) {} // 离开节点时调用
                    }
                })
        ```
        而且同一个 visitor 函数可以用于多个 AST 节点的处理，方式是指定一系列 AST，用 | 连接：
        ``` js
            // 进入 FunctionDeclaration 和 VariableDeclaration 节点时调用
            traverse(ast, {
                'FunctionDeclaration|VariableDeclaration'(path, state) {}
            })
        ```
        此外，AST 还有别名的，比如各种 XxxStatement 有个 Statement 的别名，各种 XxxDeclaration 有个 Declaration 的别名，那自然可以通过别名来指定对这些 AST 的处理：
        ``` js
                // 通过别名指定离开各种 Declaration 节点时调用
                traverse(ast, {
                    Declaration: {
                        exit(path, state) {}
                    }
                })
        ```
    - @babel/types 单个创建AST 
    - @babel/template  批量创建AST
    - @babel/generator
        ``` js
            function (ast: Object, opts: Object, code: string): {code, map} 

            //生成 sourcemap
            import generate from "@babel/generator";

            const { code, map } = generate(ast, { sourceMaps: true })
        ```
    - @babel/code-frame
    
