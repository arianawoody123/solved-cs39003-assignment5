Download Link: https://assignmentchef.com/product/solved-cs39003-assignment5
<br>
In this assignment you are required to write the semantic actions in Bison to translate a tinyC program into an array of 3-address quad’s, a supporting symbol table, and other auxiliary data structures. The translation should be machineindependent, yet it has to carry enough information so that you can later target it to a specific architecture (viz. x86-32 / x86-64).

<h1>2           Scope of Machine-Independent Translation</h1>

You may consider the following from the different phases to write actions for translation.

<h2>2.1         Expression Phase</h2>

You should support all arithmetic, shift, relational, bit, logical (boolean), and assignment expressions excluding:

<ol>

 <li><strong>sizeof </strong>operator</li>

 <li>Comma (<strong>,</strong>) operator</li>

 <li>Compound assignment operators:</li>

</ol>

*= /= %= += -= &lt;&lt;= &gt;&gt;= &amp;= ^= |=

Only simple assignment operator (=) needs to be supported.

<ol start="4">

 <li>Structure component expression</li>

</ol>

<h2>2.2         Declarations Phase</h2>

Support for declarations should be provided as follows:

<ol>

 <li>Simple variable, pointer, array, and function declarations should be supported. For example, the following would be translated:</li>

</ol>

float d = 2.3; int i, w[10]; int a = 4, *p, b; void func(int i, float d); char c;

<ol start="2">

 <li>Consider only <strong>void</strong>, <strong>char</strong>, <strong>int</strong>, and <strong>float </strong><em>type-specifiers</em>. As specified in C, <strong>char </strong>and <strong>int </strong>are to be taken as signed.</li>

</ol>

For computation of offset and storage mapping of variables, assume the following sizes<a href="#_ftn1" name="_ftnref1"><sup>[1]</sup></a> (in bytes) of types:

<table width="301">

 <tbody>

  <tr>

   <td width="57"><strong>Type</strong></td>

   <td width="71"><strong>Size</strong></td>

   <td width="173"><strong>Remarks</strong></td>

  </tr>

  <tr>

   <td width="57"><strong>void</strong></td>

   <td width="71"><em>undefined</em></td>

   <td width="173"> </td>

  </tr>

  <tr>

   <td width="57"><strong>char</strong></td>

   <td width="71">1</td>

   <td width="173"> </td>

  </tr>

  <tr>

   <td width="57"><strong>int</strong></td>

   <td width="71">4</td>

   <td width="173"> </td>

  </tr>

  <tr>

   <td width="57"><strong>float</strong></td>

   <td width="71">8</td>

   <td width="173"> </td>

  </tr>

  <tr>

   <td width="57"><strong>void *</strong></td>

   <td width="71">4</td>

   <td width="173">All pointers have same size</td>

  </tr>

 </tbody>

</table>

It may also help to support an implicit <strong>bool </strong>(boolean) type with constants <strong>1 </strong>(TRUE) and <strong>0 </strong>(FALSE). This type may be inferred for a logical expression or for an <strong>int </strong>expression in logical context. Note that the users cannot define, load, or store variables of <strong>bool </strong>type explicitly, hence it is not storable and does not have a size.

<ol start="3">

 <li>Initialization of arrays may not be considered.</li>

 <li><em>storage-class-specifier</em>, <em>enum-specifier</em>, <em>type-qualifier</em>, and <em>function-specifier </em>may not be considered.</li>

 <li>Function declaration with only parameter type list may not be considered.</li>

</ol>

Thus,

void func(int i, float d); should be supported while

void func(int, float); may not be supported.

<h2>2.3         Statement Phase</h2>

All statements excluding the following should be supported.

<ol>

 <li>Declaration within <strong>for</strong>.</li>

 <li>All Labelled statements (<em>labeled-statement</em>).</li>

 <li><strong>switch </strong>in <em>selection-statement</em>.</li>

 <li>All Jump statements (<em>jump-statement</em>) except <strong>return</strong>.</li>

</ol>

<strong>2.4       External Definitions Phase</strong>

You should support function definitions, and skip external declarations.

<h1>3           The 3-Address Code</h1>

Use the 3-Address Code specification as discussed in the class. For easy reference the same is reproduced here. Every 3-Address Code:

<ul>

 <li>Uses only up to 3 addresses.</li>

 <li>Is represented by a quad comprising – opcode, argument 1, argument 2, and result; where argument 2 is optional.</li>

</ul>

<h2>3.1         Address Types</h2>

<ul>

 <li><em>Name</em>: Source program names appear as addresses in 3-Address Codes.</li>

 <li><em>Constant</em>: Many different types and their (implicit) conversions are allowed as deemed addresses.</li>

 <li><em>Compiler-Generated Temporary</em>: Create a distinct name each time a temporary is needed – good for optimization.</li>

</ul>

<h2>3.2         Instruction Types</h2>

For Addresses x, y, z, and Label L, the following instruction types should be supported:

<ul>

 <li><em>Binary Assignment Instruction</em>: For a binary op (including arithmetic, shift, relational, bit, or logical operators):</li>

</ul>

x = y op z

<ul>

 <li><em>Unary Assignment Instruction</em>: For a unary operator op (including unary minus or plus, logical negation, bit, and conversion operators):</li>

</ul>

x = op y

<ul>

 <li><em>Copy Assignment Instruction</em>:</li>

</ul>

x = y

<ul>

 <li><em>Unconditional Jump</em>: goto L</li>

 <li><em>Conditional Jump</em>:

  <ol>

   <li><em>Value-based</em>:</li>

  </ol></li>

</ul>

if x goto L ifFalse x goto L

<ol>

 <li><em>Comparison-based</em>: For a relational operator op (including <em>&lt;</em>, <em>&gt;</em>, ==, ! =, ≤, ≥):</li>

</ol>

if x relop y goto L

<ul>

 <li><em>Function Call</em>: A function call p(x1, x2, …, xN) having N≥ 0 parameters is coded as (for addresses p, x1, x2, and xN):</li>

</ul>

param x1 param x2 … param xN

y = call p, N

Note that N is not redundant as procedure calls can be nested.

<ul>

 <li><em>Return Value</em>: Returning a value and / or assigning it is optional. If there is a return value v it is returned from the procedure p as:</li>

</ul>

return v

<ul>

 <li><em>Indexed Copy Instructions</em>: It can be used as:</li>

</ul>

x = y[z] x[z] = y

<ul>

 <li><em>Address and Pointer Assignment Instructions</em>: These can be used as:</li>

</ul>

x = &amp;y x = *y *x = y

<h1>4           Design of the Translator</h1>

<ol start="2">

 <li><strong>Lexer &amp; Parser</strong>: Use the Flex and Bison specifications (if required you may correct your specifications) you had developed in Assignments 3 and 4 respectively, and write semantic actions for translating the subset of tinyC as specified in Section 2. Note that many grammar rules of your tinyC parser may not have any action or may just have propagate-only actions. Also, some of the lexical tokens may not be used.</li>

 <li><strong>Augmentation</strong>: Augment the grammar rules with markers and add new grammar rules as needed for the intended semantic actions. Justify your augmentation decisions within comments of the rules.</li>

 <li><strong>Attributes</strong>: Design the attributes for every grammar symbol (terminal as well as non-terminal). List the attributes against symbols (with brief justification) in comment on the top of your Bison specification file. Highlight the inherited attributes, if any.</li>

 <li><strong>Symbol Table</strong>: Use symbol tables for user-defined (including arrays and pointers) variables, temporary variables and functions. The symbol table should have at least the following fields:</li>

</ol>

<table width="322">

 <tbody>

  <tr>

   <td width="55"><strong>Name</strong></td>

   <td width="50"><strong>Type</strong></td>

   <td width="56"><strong>Initial</strong></td>

   <td width="42"><strong>Size</strong></td>

   <td width="55"><strong>Offset</strong></td>

   <td width="62"><strong>Nested</strong></td>

  </tr>

  <tr>

   <td width="55"> </td>

   <td width="50"> </td>

   <td width="56"><strong>Value</strong></td>

   <td width="42"> </td>

   <td width="55"> </td>

   <td width="62"><strong>Table</strong></td>

  </tr>

  <tr>

   <td width="55">…</td>

   <td width="50">…</td>

   <td width="56">…</td>

   <td width="42">…</td>

   <td width="55">…</td>

   <td width="62">…</td>

  </tr>

 </tbody>

</table>

For example, for

float d = 2.3; int i, w[10]; int a = 4, *p, b; void func(int i, float d); char c; the Symbol Tables will look like:

<strong>ST(global)                                        </strong><em>This is the Symbol Table for global symbols</em>

<table width="441">

 <tbody>

  <tr>

   <td width="55"><strong>Name</strong></td>

   <td width="111"><strong>Type</strong></td>

   <td width="56"><strong>Initial</strong></td>

   <td width="42"><strong>Size</strong></td>

   <td width="55"><strong>Offset</strong></td>

   <td width="121"><strong>Nested</strong></td>

  </tr>

  <tr>

   <td width="55"> </td>

   <td width="111"> </td>

   <td width="56"><strong>Value</strong></td>

   <td width="42"> </td>

   <td width="55"> </td>

   <td width="121"><strong>Table</strong></td>

  </tr>

  <tr>

   <td width="55">d</td>

   <td width="111"><strong>float</strong></td>

   <td width="56">2.3</td>

   <td width="42">8</td>

   <td width="55">0</td>

   <td width="121">null</td>

  </tr>

  <tr>

   <td width="55">i</td>

   <td width="111"><strong>int</strong></td>

   <td width="56">null</td>

   <td width="42">4</td>

   <td width="55">8</td>

   <td width="121">null</td>

  </tr>

  <tr>

   <td width="55">w</td>

   <td width="111"><em>array</em>(10, <strong>int</strong>)</td>

   <td width="56">null</td>

   <td width="42">40</td>

   <td width="55">12</td>

   <td width="121">null</td>

  </tr>

  <tr>

   <td width="55">a</td>

   <td width="111"><strong>int</strong></td>

   <td width="56">4</td>

   <td width="42">4</td>

   <td width="55">52</td>

   <td width="121">null</td>

  </tr>

  <tr>

   <td width="55">p</td>

   <td width="111"><em>ptr</em>(<strong>int</strong>)</td>

   <td width="56">null</td>

   <td width="42">4</td>

   <td width="55">56</td>

   <td width="121">null</td>

  </tr>

  <tr>

   <td width="55">b</td>

   <td width="111"><strong>int</strong></td>

   <td width="56">null</td>

   <td width="42">4</td>

   <td width="55">60</td>

   <td width="121">null</td>

  </tr>

  <tr>

   <td width="55">func</td>

   <td width="111"><em>function</em></td>

   <td width="56">null</td>

   <td width="42">0</td>

   <td width="55">64</td>

   <td width="121">ptr-to-ST(func)</td>

  </tr>

  <tr>

   <td width="55">c</td>

   <td width="111"><strong>char</strong></td>

   <td width="56">null</td>

   <td width="42">1</td>

   <td width="55">64</td>

   <td width="121">null</td>

  </tr>

 </tbody>

</table>

<strong>ST(func)                     </strong><em>This is the Symbol Table for function </em>func

<table width="373">

 <tbody>

  <tr>

   <td width="58"><strong>Name</strong></td>

   <td width="50"><strong>Type</strong></td>

   <td width="56"><strong>Initial</strong></td>

   <td width="42"><strong>Size</strong></td>

   <td width="55"><strong>Offset</strong></td>

   <td width="111"><strong>Nested</strong></td>

  </tr>

  <tr>

   <td width="58"> </td>

   <td width="50"> </td>

   <td width="56"><strong>Value</strong></td>

   <td width="42"> </td>

   <td width="55"> </td>

   <td width="111"><strong>Table</strong></td>

  </tr>

  <tr>

   <td width="58">i</td>

   <td width="50"><strong>int</strong></td>

   <td width="56">null</td>

   <td width="42">4</td>

   <td width="55">0</td>

   <td width="111">null</td>

  </tr>

  <tr>

   <td width="58">d</td>

   <td width="50"><strong>float</strong></td>

   <td width="56">null</td>

   <td width="42">8</td>

   <td width="55">4</td>

   <td width="111">null</td>

  </tr>

  <tr>

   <td width="58">retVal</td>

   <td width="50"><strong>void</strong></td>

   <td width="56">null</td>

   <td width="42">0</td>

   <td width="55">12</td>

   <td width="111">null</td>

  </tr>

 </tbody>

</table>

The Symbol Tables should support the following methods:

<table width="418">

 <tbody>

  <tr>

   <td width="100">lookup(…)</td>

   <td width="318">A method to lookup an id (given its name or lexeme) in the Symbol Table. If the id exists, the entry is returned, otherwise a new entry is created.</td>

  </tr>

  <tr>

   <td width="100">gentemp(…)</td>

   <td width="318">A static method to generate a new temporary, insert it to the Symbol Table, and return a pointer to the entry.</td>

  </tr>

  <tr>

   <td width="100">update(…)</td>

   <td width="318">A method to update different fields of an existing entry.</td>

  </tr>

  <tr>

   <td width="100">print(…)</td>

   <td width="318">A method to print the Symbol Table in a suitable format.</td>

  </tr>

 </tbody>

</table>

<strong>Points to Note</strong>:

<ul>

 <li>The fields and the methods are indicative. You may change their name, functionality and also add other fields and / or methods that you may need.</li>

 <li>It should be easy to extend the Symbol Table as further features are supported and more functionality is added.</li>

 <li>The global symbol table is unique.</li>

 <li>Every function will have a symbol table of its parameters and automatic variables. This symbol table will be nested in the global symbol table.</li>

 <li>Symbol definitions within blocks are naturally carried in separate symbol tables. Each such table will be nested in the symbol table of the enclosing scope. This will give rise to an implicit stack of symbol tables (global one being the bottom-most) the while symbols are processed during translation. The search for a symbol starts from the top-most (current) table and goes down the stack up to the global table.</li>

</ul>

<ol>

 <li>e) Quad <strong>Array</strong>: The array to store the 3-address quad’s. Index of a quad in the array is the <em>address </em>of the 3-address code. The quad array will have the following fields (having usual meanings):</li>

</ol>

<table width="186">

 <tbody>

  <tr>

   <td width="32"><strong>op</strong></td>

   <td width="50"><strong>arg 1</strong></td>

   <td width="50"><strong>arg 2</strong></td>

   <td width="54"><strong>result</strong></td>

  </tr>

  <tr>

   <td width="32">…</td>

   <td width="50">…</td>

   <td width="50">…</td>

   <td width="54">…</td>

  </tr>

 </tbody>

</table>

<em>Note</em>:

<ul>

 <li><strong>arg 1 </strong>and / or <strong>arg 2 </strong>may be a variable (address) or a constant.</li>

 <li><strong>result </strong>is variable (address) only.</li>

 <li><strong>arg 2 </strong>may be null.</li>

</ul>

For example, if

int i = 10, a[10], v = 5;

…

do i = i – 1; while (a[i] &lt; v);

translates to

100: t1 = i – 1 101: i = t1

102: t2 = i * 4

103: t3 = a[t2]

104: if t3 &lt; v goto 100

the quad’s are represented as:

<table width="253">

 <tbody>

  <tr>

   <td width="55"><strong>Index</strong></td>

   <td width="44"><strong>op</strong></td>

   <td width="50"><strong>arg 1</strong></td>

   <td width="50"><strong>arg 2</strong></td>

   <td width="54"><strong>result</strong></td>

  </tr>

  <tr>

   <td width="55">…</td>

   <td width="44">…</td>

   <td width="50">…</td>

   <td width="50">…</td>

   <td width="54">…</td>

  </tr>

  <tr>

   <td width="55">100</td>

   <td width="44">–</td>

   <td width="50">i</td>

   <td width="50">1</td>

   <td width="54">t1</td>

  </tr>

  <tr>

   <td width="55">101</td>

   <td width="44">=</td>

   <td width="50">t1</td>

   <td width="50"> </td>

   <td width="54">i</td>

  </tr>

  <tr>

   <td width="55">102</td>

   <td width="44">*</td>

   <td width="50">i</td>

   <td width="50">4</td>

   <td width="54">t2</td>

  </tr>

  <tr>

   <td width="55">103</td>

   <td width="44">=[ ]</td>

   <td width="50">a</td>

   <td width="50">t2</td>

   <td width="54">t3</td>

  </tr>

  <tr>

   <td width="55">104</td>

   <td width="44"><em>&lt;</em></td>

   <td width="50">t3</td>

   <td width="50">v</td>

   <td width="54">100</td>

  </tr>

 </tbody>

</table>

The Quad Array should support the following methods:

<table width="404">

 <tbody>

  <tr>

   <td width="86">emit(…)</td>

   <td width="318">An overloaded static method to add a (newly generated) quad of the form: result = arg1 op arg2 where op usually is a binary operator. If arg2 is missing, op is unary. If op also is missing, this is a copy instruction.</td>

  </tr>

  <tr>

   <td width="86">print(…)</td>

   <td width="318">A method to print the quad array in a suitable format.</td>

  </tr>

 </tbody>

</table>

For example the above state of the array may be printed (with the symbol information) as:

void main()

{

int i = 10; int a[10]; int v = 5; int t1; int t2; int t3;

L100: t1 = i – 1; L101: i = t1;

L102: t2 = i * 4;

L103: t3 = a[t2];

L104: if (t3 &lt; v) goto L100;

}

<em>Point to Note</em>:

<ul>

 <li>The fields and the methods are indicative. You may change their name, functionality and also add other fields and / or methods that you may need.</li>

</ul>

<ol>

 <li>f) <strong>Global Functions</strong>: Following (or similar) global functions and more may be needed to implement the semantic actions:</li>

</ol>

<table width="423">

 <tbody>

  <tr>

   <td width="423">makelist(i)A global function to create a new list containing only i, an index into the array of quad’s, and to return a pointer to the newly created list.</td>

  </tr>

  <tr>

   <td width="423">merge(p1, p2)A global function to concatenate two lists pointed to by p1 and p2 and to return a pointer to the concatenated list.</td>

  </tr>

  <tr>

   <td width="423">backpatch(p, i)A global function to insert i as the target label for each of the quad’s on the list pointed to by p.</td>

  </tr>

  <tr>

   <td width="423">typecheck(E1, E2)A global function to check if E1 &amp; E2 have same types (that is, if &lt;type of E1&gt; = &lt;type of E2&gt;). If not, then to check if they have compatible types (that is, one can be converted to the other), to use an appropriate conversion function conv&lt;type of E1&gt;2&lt;type of E2&gt;(E) or conv&lt;type of E2&gt;2&lt;type of E1&gt;(E) and to make the necessary changes in the Symbol Table entries. If not, that is, they are of incompatible types, to throw an exception during translation.</td>

  </tr>

  <tr>

   <td width="423">conv&lt;type1&gt;2&lt;type2&gt;(E)A global function to convert<em><sup>a </sup></em>an expression E from its current type type1 to target type type2, to adjust the attributes of E accordingly, and finally to generate additional codes, if needed.</td>

  </tr>

 </tbody>

</table>

<em><sup>a</sup></em>It is assumed that this function is called from typecheck(E1, E2) and hence the

conversion is possible.

Naturally, these are indicative and should be adopted as needed. For every function used clearly explain the input, the output, the algorithm, and the purpose with possible use at the top of the function.

<h1>5           The Assignment</h1>

<ol>

 <li>Write a 3-Address Code translator based on the Flex and Bison specifications of tinyC. Assume that the input tinyC file is lexically, syntactically, and semantically correct. Hence no error handling and / or recovery is expected.</li>

 <li>Prepare a Makefile to compile and test the project.</li>

 <li>Prepare test input files ass5 <em>roll </em>test<em>&lt;</em>number<em>&gt;</em>.c to test the semantic actions and generate the translation output in ass5 <em>roll </em>quads<em>&lt;</em>number<em>&gt;</em>.out.</li>

 <li>Name your files as follows:</li>

</ol>

<table width="436">

 <tbody>

  <tr>

   <td width="243"><strong>File</strong></td>

   <td width="193"><strong>Naming</strong></td>

  </tr>

  <tr>

   <td width="243">Flex Specification</td>

   <td width="193">ass5 <em>roll</em>.l</td>

  </tr>

  <tr>

   <td width="243">Bison Specification</td>

   <td width="193">ass5 <em>roll</em>.y</td>

  </tr>

  <tr>

   <td width="243">Data Structures (Class Definitions) &amp;Global Function Prototypes</td>

   <td width="193">ass5 <em>roll </em>translator.h</td>

  </tr>

  <tr>

   <td width="243">Data Structures, Function Implementations &amp; Translator main()</td>

   <td width="193">ass5 <em>roll </em>translator.cxx</td>

  </tr>

  <tr>

   <td width="243">Test Inputs</td>

   <td width="193">ass5 <em>roll </em>test<em>&lt;</em>number<em>&gt;</em>.c</td>

  </tr>

  <tr>

   <td width="243">Test Outputs</td>

   <td width="193">ass5 <em>roll </em>quads<em>&lt;</em>number<em>&gt;</em>.out</td>

  </tr>

 </tbody>

</table>

<ol start="5">

 <li>Prepare a tar-archive with the name ass5 <em>roll</em>.tar containing all the files and upload to Moodle.</li>

</ol>

<h1>6           Credits</h1>

Design of Grammar Augmentations:                                                                                    <strong>5</strong>

<em>Explain the augmentations in the production rules in Bison</em>

Design of Attributes:                                                                                                                 <strong>5</strong>

<em>Explain the attributes in the respective </em>%token <em>and </em>%type <em>in Bison</em>

Design and Implementation of Symbol Table &amp; Supporting

Data Structures:                                                                                                                       <strong>10</strong>

<em>Explain with class definition of ST &amp; other Data Structures</em>

Design and Implementation of Quad Array:                                                                       <strong>5</strong>

<em>Explain with class definition of QA</em>

Design and Implementation of Global Functions:                                                          <strong>10</strong>

<em>Explain i/p, o/p, algorithm &amp; purpose for every function</em>

Design and Implementation of Semantic Actions:

<em>Explain with every action in Bison</em>

Expression Phase:                                                                                                       <strong>15</strong>

<em>Correct handling of operators, type checking &amp; conversions</em>

Declaration Phase:                                                                                                      <strong>10</strong>

<em>Handling of variable declarations, function definitions in ST</em>

Statement Phase:                                                                                                         <strong>15</strong>

<em>Correct handling of statements</em>

External Definition Phase:                                                                                          <strong>5</strong>

<em>Correct handling of function definitions</em>

Design of Test files and correctness of outputs:                                                             <strong>20</strong>

<em>Test at least 5 input files covering all rules</em>

<em>Shortcoming and / or bugs, if any, should be highlighted</em>

<a href="#_ftnref1" name="_ftn1">[1]</a> Using hard-coded sizes for types does not keep the code machine-independent. Hence you may want to use constants like size of char, size of int, size of float, and size of pointer for sizes that can be defined at the time of machine-dependent targeting.