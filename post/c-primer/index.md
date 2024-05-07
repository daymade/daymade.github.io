
<!--more-->

```c
	#include "stdio.h"
	#include "math.h"

	void main()
	{
		for (int n = 0; n < 100; n++)
		{
			if (isPrimer(n))
			{
				printf("%d\n", n);
			}
		}
		system("pause");
	}
	
	isPrimer(int n)
	{
		for (int i = 2; i <= sqrt(n); i++)
		{
			if (n % i == 0)
			{
				return 0;
			}
		}
	}
```