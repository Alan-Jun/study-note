| Modifier and Type | Method and Description                                       |
| ----------------- | ------------------------------------------------------------ |
| static double     | **abs(double a)**  返回值为 double绝对值。                   |
| static float      | **abs(float a)**  返回 float值的绝对值。                     |
| static int        | **abs(int a)**  返回值为 int绝对值。                         |
| static long       | **abs(long a)**  返回值为 long绝对值。                       |
| static int        | **incrementExact(int a)**  将 a 加1, 如果溢出int则抛出异常<br/>ArithmeticException("integer overflow"); |
| static long       | **incrementExact(long a)**  将 a 加1, 如果溢出则抛出异常，如果结果溢出long，则抛出异常<br/>ArithmeticException("long overflow"); |
| static int        | **decrementExact(int a)**  返回一个递减1的参数，如果结果溢出int，<br/>ArithmeticException("integer overflow"); |
| static long       | **decrementExact(long a)**  将返回的参数递减1，如果结果溢出long，则抛出异常<br/>ArithmeticException("long overflow"); |
| static int        | **addExact(int x,  int y)** 返回其参数的总和，如果结果溢出int，则抛出异常<br/> ArithmeticException("integer overflow") |
| static long       | **addExact(long x,  long y)**  返回其参数的总和，如果结果溢出long，则抛出异常<br/>ArithmeticException("long overflow"); |
| static double     | **ceil(double a)**  返回大于或等于参数a的最小（最接近负无穷大） double值，等于一个数学整数。 |
| static double     | **floor(double a)**  返回小于或等于参数a的最大（最接近正无穷大） double值，**等于一个数学整数**。<br/> Math.floor(60984.1)=60984.0<br/> Math.floor(-497.99)=-498.0<br/> Math.floor(0)=0.0 |
| static int        | **floorDiv(int x,  int y)**  返回 **floor( x / y ）** int值  |
| static long       | **floorDiv(long x,  long y)**  返回**floor( x / y ）**  long值。 |
| static int        | **floorMod(int x,  int y)**  返回 **x - floorDiv(x, y) * y;** |
| static long       | **floorMod(long x,  long y)**  返回 **x - floorDiv(x, y) * y;** |
| static double     | **copySign(double magnitude,  double sign)**  使用第二个浮点参数的符号（正负数符号）返回第一个浮点参数。<br/> Math.copySign(125.9, -0.4873)=-125.9<br/> Math.copySign(-0.4873, 125.9)=0.4873 |
| static float      | **copySign(float magnitude,  float sign)**  使用第二个浮点参数的符号（正负数符号）返回第一个浮点参数。 |
| static double     | **exp(double a)**  返回欧拉数e为底的双精度值的 a 次幂== e[^a] |
| static double     | **expm1(double x)**  返回 *e* [^x] -1。的双精度值            |
| static double     | **sqrt(double a)**  返回根号下a , 如果参数小于零，则结果为NaN。<br/> sqrt(9) = 3<br/> sqrt(25) = 5<br/> sqrt(-25) = NaN |
| static double     | **hypot(double x,  double y)**  返回 sqrt（ x[^2] + y[^2]  ），没有中间溢出或下溢。 |
| static double     | **IEEEremainder(double f1,  double f2)**  根据IEEE 754标准计算f1%f2取余数运算。 |
| static double     | **log(double a)**  返回的自然对数（以 *e*为底） double值。 ln(x ) |
| static double     | **log10(double a)**  返回一个 double的基数10对数值。         |
| static double     | **log1p(double x)**  返回参数和1的和的自然对数。ln(x + 1)    |
| static double     | **max(double a,  double b)**  返回两个 double值中的较大值。  |
| static float      | **max(float a,  float b)**  返回两个 float的较大值。         |
| static int        | **max(int a,  int b)**  返回两个 int值中的较大值。           |
| static long       | **max(long a,  long b)**  返回两个 long的较大值。            |
| static double     | **min(double a,  double b)**  返回两个 double的较小值。      |
| static float      | **min(float a,  float b)**  返回两个 float的较小值。         |
| static int        | **min(int a,  int b)**  返回两个 int的较小值。               |
| static long       | **min(long a,  long b)**  返回两个 long的较小值。            |
| static int        | **multiplyExact(int x,  int y)**  返回参数的乘积，如果结果溢出int，则抛出 <br/>ArithmeticException("integer overflow"); |
| static long       | **multiplyExact(long x,  long y)**  返回参数的乘积，如果结果溢出long，则抛出<br/>ArithmeticException("long overflow"); |
| static int        | **negateExact(int a)**  返回 -a ，如果结果溢出int，则<br/>ArithmeticException("integer overflow"); |
| static long       | **negateExact(long a)**  返回参数的否定，如果结果溢出long，则<br/>ArithmeticException("long overflow"); |
| static double     | **pow(double a,  double b)**  返回 a[^b]                     |
| static double     | **random()**  返回值为 double值为正号，大于等于 0.0 ，小于  1.0 。 |
| static double     | rint(double a)  返回与参数最接近值的 double值，并且等于数学整数。 |
| static long       | round(double a)  返回参数中最接近的 long ，其中 long四舍五入为正无穷大。 |
| static int        | round(float a)  返回参数中最接近的 int ，其中 int四舍五入为正无穷大。 |
| static double     | scalb(double d,  int scaleFactor)  返回 d 2  scaleFactor四舍五入，<br/>好像由单个正确四舍五入的浮点乘以双重值集合的成员执行。 |
| static float      | scalb(float f,  int scaleFactor)  返回 f 2  scaleFactor四舍五入，<br/>就像一个正确圆形的浮点数乘以浮点值集合的成员一样。 |
| static double     | signum(double d)  返回参数的signum函数;  如果参数为零，则为零，<br/>如果参数大于零则为1.0，如果参数小于零，则为-1.0。 |
| static float      | signum(float f)  返回参数的signum函数;  如果参数为零，则为零，<br/>如果参数大于零则为1.0f，如果参数小于零，则为-1.0f。 |
| static int        | subtractExact(int x,  int y)  返回参数的差异，如果结果溢出int，则抛出 int 。 |
| static long       | subtractExact(long x,  long y)  返回参数的差异，如果结果溢出long，则抛出 long 。 |
| static int        | toIntExact(long value)  返回long参数的值;  如果值溢出int，则int 。 |
| static double     | toRadians(double angdeg)  将以度为单位的角度转换为以弧度测量的大致相等的角度。 |
| static double     | ulp(double d)  返回参数的ulp的大小。                         |
| static float      | ulp(float f)  返回参数的ulp的大小。                          |
| static double     | asin(double a)  返回值的正弦值; 返回角度在*pi*/ 2到*pi*/  2的范围内。 |
| static double     | atan(double a)  返回值的反正切值; 返回角度在*pi*/ 2到*pi*/  2的范围内。 |
| static double     | atan2(double y,  double x)  返回从直角坐标（转换角度 *theta* x ， y ）为极坐标  *（R，θ-）。* |
| static double     | acos(double a)  返回值的反余弦值; 返回的角度在0.0到*pi*的范围内。 |
| static double     | cos(double a)  返回角度的三角余弦。                          |
| static double     | cosh(double x)  返回的双曲余弦 double值。                    |
| static double     | sin(double a)  返回角度的三角正弦。                          |
| static double     | sinh(double x)  返回的双曲正弦 double值。                    |
| static double     | tan(double a)  返回角度的三角正切。                          |
| static double     | tanh(double x)  返回的双曲正切 double值。                    |
| static double     | toDegrees(double angrad)  将以弧度测量的角度转换为以度为单位的近似等效角度。 |



| static double     | **cbrt(double a)**  返回 double值的多维数据集根。            |
| static int        | getExponent(double d)  返回d的表示中使用的无偏指数 double 。 |
| static int        | getExponent(float f)  返回a的表示中使用的无偏指数 float 。   |
| static double     | nextAfter(double start,  double direction)  返回与第二个参数方向相邻的第一个参数的浮点数。 |
| static float      | nextAfter(float start,  double direction)  返回与第二个参数方向相邻的第一个参数的浮点数。 |
| static double     | nextDown(double d)  返回与负无穷大方向相邻的 d的浮点值。     |
| static float      | nextDown(float f)  返回与负无穷大方向相邻的 f的浮点值。      |
| static double     | nextUp(double d)  返回与正无穷大方向相邻的 d的浮点值。       |
| static float      | nextUp(float f)  返回与正无穷大方向相邻的 f的浮点值。        |