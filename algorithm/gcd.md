辗转相除法基于如下原理：两个整数的最大公约数等于其中较小的数和两数的相除余数的最大公约数。
 
```
     long gcd(long a, long b) {
        if(b==0) {
            return a;
        }else{
            return gcd(b, a % b);
        }
    }
 ```
