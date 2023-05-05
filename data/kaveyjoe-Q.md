target:https://github.com/code-423n4/2023-05-ajna/blob/main/ajna-grants/src/grants/libraries/Maths.sol

Summary:
this contract contains some issues related to the use of fixed-point arithmetic. The use of a constant value of 10^9 as the fixed-point representation in the wsqrt function could lead to inaccuracies in the calculation of the square root, depending on the input value. Additionally, the use of the constant value of 10^18 in the wpow, wmul, and wdiv functions could also lead to inaccuracies in calculations involving large values.

Severity:
Low

Description:
The wsqrt function uses a constant value of 10^9 to represent fixed-point numbers. While this may work well for some values, it could result in inaccuracies for other values, depending on the precision required. For example, if the input value is very small, the square root may be rounded to zero, as the result is multiplied by 10^9. On the other hand, if the input value is very large, the result may be truncated, as the square root is divided by 10^9. To ensure accurate calculations, it may be better to use a variable representing the number of decimal places, rather than a fixed constant.

Similarly, the wpow, wmul, and wdiv functions use a constant value of 10^18 as the fixed-point representation. While this may be sufficient for most calculations, it could result in inaccuracies for very large numbers. For example, if the result of a multiplication exceeds 10^36, the calculation could overflow. To avoid these issues, it may be better to use a variable representing the number of decimal places, rather than a fixed constant.

Recommendation:
To ensure accurate calculations, it is recommended to use a variable representing the number of decimal places, rather than a fixed constant, for all fixed-point arithmetic calculations. This would allow for greater precision and avoid issues with overflow or truncation.