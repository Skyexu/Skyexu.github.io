---
title: HDUACM1200
date: 2016/9/21 15:00:56   
tags: ACM
categories: ACM

---
### 题目：

```
To and Fro

Time Limit: 2000/1000 MS (Java/Others)    Memory Limit: 65536/32768 K (Java/Others)
Total Submission(s): 6390    Accepted Submission(s): 4389


Problem Description
Mo and Larry have devised a way of encrypting messages. They first decide secretly on the number of columns and write the message (letters only) down the columns, padding with extra random letters so as to make a rectangular array of letters. For example, if the message is “There’s no place like home on a snowy night” and there are five columns, Mo would write down

t o i o y
h p k n n
e l e a i
r a h s g
e c o n h
s e m o t
n l e w x


Note that Mo includes only letters and writes them all in lower case. In this example, Mo used the character ‘x’ to pad the message out to make a rectangle, although he could have used any letter.

Mo then sends the message to Larry by writing the letters in each row, alternating left-to-right and right-to-left. So, the above would be encrypted as

toioynnkpheleaigshareconhtomesnlewx

Your job is to recover for Larry the original message (along with any extra padding letters) from the encrypted one.
 

Input
There will be multiple input sets. Input for each set will consist of two lines. The first line will contain an integer in the range 2. . . 20 indicating the number of columns used. The next line is a string of up to 200 lower case letters. The last input set is followed by a line containing a single 0, indicating end of input.
 

Output
Each input set should generate one line of output, giving the original plaintext message, with no spaces.
 

Sample Input
5
toioynnkpheleaigshareconhtomesnlewx
3
ttyohhieneesiaabss
0
 

Sample Output
theresnoplacelikehomeonasnowynightx
thisistheeasyoneab
 

Source
East Central North America 2004
 

Recommend
Ignatius.L   |   We have carefully selected several similar problems for you:  1196 1073 1161 1113 1256
```
### 解答：
```
package hdu;

import java.util.Scanner;
/**
 * 做的时候没看清楚  直接把蛇形输出做了正序的转换，然后才进行二维数组存储，多了个步骤
 * @author Skye
 *
 */
public class Main {

	public static void main(String[] args) {
		// TODO Auto-generated method stub

		Scanner sc = new Scanner(System.in);
		int col = sc.nextInt();
		String message = null;
		int rowNum,row = 0;
		StringBuffer result = new StringBuffer();
		char[] stringArr = null;
		StringBuffer tmpString = new StringBuffer();
		char[] tmpChar = null;
		boolean isChange = false;
		while (col != 0) {
			message = sc.next();
			rowNum = message.length()/col;
			// System.out.println(message);
			stringArr = message.toCharArray();
			// ttyohhieneesiaabss
			for (int i = 0; i < stringArr.length; i++) {
				// System.out.println(stringArr[i]);
				if (i % col == 0) { // 到行开头
					row = i / col + 1; // 记录当前字母在第几行
					if (row % 2 == 0) {
						isChange = true;
					} else {
						isChange = false;
					}
				}

				if (isChange == false) {
					
					if (i != 0 && i % col == 0) {
						//System.out.println(tmpString.toString());
						tmpChar = tmpString.toString().toCharArray();
						for (int j = tmpChar.length - 1; j >= 0; j--) {
							result.append(tmpChar[j]);
						}
						tmpString = new StringBuffer();
					}
					result.append(stringArr[i]);
				} else {
					tmpString.append(stringArr[i]);
				}

			}
			if(isChange == true){
				//System.out.println(tmpString.toString());
				tmpChar = tmpString.toString().toCharArray();
				for (int j = tmpChar.length - 1; j >= 0; j--) {
					result.append(tmpChar[j]);
				}
				tmpString = new StringBuffer();
			}
			//System.out.println(result);
			
			char[][] charArr = new char[rowNum][col];
			char[] resultChar = result.toString().toCharArray();
			int count = 0;
			for(int k = 0; k < rowNum;k++){
				for(int p = 0; p < col;p++){
					charArr[k][p] = resultChar[count++];
				}
			}
			StringBuffer resultFinal = new StringBuffer();
			for(int k = 0; k < col;k++){
				for(int p = 0; p < rowNum;p++){
					resultFinal.append(charArr[p][k]);
				}
			}
			System.out.println(resultFinal);
			resultFinal = new StringBuffer();
			result = new StringBuffer();
			col = sc.nextInt();
		}
		sc.close();
	}

}

```

### HDU另解：
```
import java.io.*;
import java.util.*;

public class Main
{

	public static void main(String[] args)
	{
		Scanner input = new Scanner(System.in);
		while (input.hasNext())
		{
			int n = input.nextInt();
			if(n==0) break;
			input.nextLine();
			String str = input.nextLine();
			char c1[] = str.toCharArray();                   // 把每个字符单个存起来
			char c2[][] = new char[1010][1010];
			int line = c1.length / n;                        //记录行数
			int k = 0;                                      // 记录c1字符的位数
			for (int i = 0; i < line; i++)
			{
				if (i % 2 == 1)
				{
					for (int j = n - 1; j >= 0; j--)
					{
						c2[i][j] = c1[k++];
					}
				} 
				else
				{
					for (int j = 0; j < n; j++)
					{
						c2[i][j] = c1[k++];
					}
				}
			}

			for (int j = 0; j < n; j++)
			{
				for (int i = 0; i < line; i++)
				{
					System.out.print(c2[i][j]);
				}
			}
			System.out.println();
		}
	}

}
```