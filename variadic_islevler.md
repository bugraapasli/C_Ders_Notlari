## C dilinde istenilen sayıda argümanla çağrılabilen işlevler `(variadic functions)`
Bir işlevin istenilen sayıda değerle çağrılması programlamada genel bir ihtiyaç ve programlama dillerinin çoğunda bu ihtiyacı karşılamaya yönelik araç ya da araçlar var. Özellikle giriş çıkış işlemlerine `(input-output operations)` yönelik hizmet veren kütüphanelerde böyle işlevler tercih ediliyor. Böyle işlevlere ingilizcede popüler olarak `"variadic functions"` deniyor. Ben bu terim yerine Türkçede "istenilen sayıda argüman ile çağrılabilen işlev" terimini kullanacağım. Çağrıyı yapan kod böyle işlevlere, ne yaptırmak istediğine bağlı olarak, farklı sayıda veri gönderebiliyor. Örnekler verelim:

`print` isimli bir işlev kendisine gönderilen her bir ifadenin değerini ekrana yazdırıyor olabilir.
`get_ave` isimli işlev kendisine gönderilen tam sayıların aritmetik ortalamasını hesaplıyor olabilir.
`concat` isimli işlev kendisine gönderilen yazıları birleştirip tek bir yazı oluşturuyor olabilir.
`push_back_vals` isimli işlev kendisine gönderilen değerleri bir dinamik diziye ekliyor olabilir.

Farklı programlama dillerinin böyle bir işlevin oluşturulmasını sağlayan farklı araçları var. Örneğin `C++` dilinde bu yapı için ağırlıklı olarak türden bağımsız programlama `(generic programming)` paradigmasına destek veren araçlardan faydalanılıyor. Bu yazının amacı C dilinde değişken sayıda argümanla çağrılabilen işlevleri ayrıntılı olarak incelemek.

C dilinde bir işlevin `"variadic"` olduğunu gösteren `...` atomu. Yan yana yazılmış üç nokta karakterinin oluşturduğu atoma `(token)` ingilizcede `ellipsis` deniyor. Bir işlevin `"variadic"` olması için son parametre değişkeni olarak `...` atomunun yazılması gerekiyor. üç nokta atomu atomu ile gösterilen parametreye bundan sonra `"variadic parametre"` diyeceğim.

```
void print(int n, ...);
int sum(int n, ...);
double ave(int, int, ...)
```
Bu işlevlerin çağrılmasına ilişkin genel kural şu: Çağrıyı yapan taraf `"variadic"` parametreden önce yer alan türleri belirtilmiş tüm parametre değişkenlerine argüman göndermek zorunda. `variadic` parametre için kendi seçimine bağlı olarak `(opsiyonel olarak)` dilediği sayıda argüman olarak gönderilebilir. Aşağıdaki kodu inceleyelim:

```
void func(int, int, ...);

int main()
{
	func(10, 20, 30, 40);
	func(1, 3, 5);
	func(2, 4);
	func(2); //geçersiz
	func(0); //geçersiz
}
```
`"variadic parametre"` son parametre olarak yazılmalı ve kendisinden önce türü belirtilmiş en az bir parametre değişkeni olmalı.

## tür kontrolü
Seçime bağlı olarak gönderilecek argümanların türlerinin "variadic" işlev tarafından bilinmesi mümkün değil. Derleyici seçimlik `(opsiyonel)` argümanları işleve göndermeden "varsayılan argüman dönüşümü" `(default argument conversion)` denilen dönüşümü gerçekleştirir. Yani `char` ve `short` türlerden olan argümanlar işaretli ya da işaretsiz int türüne, `float` türden olan argümanlar ise `double` türüne dönüştürülürler. İstenilen sayıda argümanla çağrılan işlevlerin en zayıf tarafı da bu. Derleyicinin bir tür kontrolü yapma şansı yok. Bu yüzden `variadic` işlevler normal işlevlere göre daha yüksek kodlama hatası riski içeriyorlar.

## stdarg.h başlık dosyası içinde tanımlanan makrolar
`variadic` bir işlev tanımlayabilmemiz için standart `<stdarg.h>` başlık dosyasında bildirilen `va_list` türünün ve yine aynı başlık dosyasında tanımlanan bazı standart makroların kullanılması gerekiyor:

## va_list
`va_list` opsiyonel argümanları gösterecek bir pointer türüne verilen bir tür eş ismi `(type alias)`. Standartlar `va_list`'in hangi türe bir `typedef` bildirimi ile eş isim olarak seçileceğini derleyicilere bırakmış. Seçimlik argümanların dolaşılabilmesi için `va_list` türünden bir değişkenin tanımlanması ve bu değişkene `va_start` makrosuyla değer verilmesi gerekiyor.

## va_start
`va_start` aşağıdaki gibi bir makro:

```
void va_start (va_list args, last_req)
```
Bu makro seçimlik argümanları dolaşma işleminde kullanılacak `va_list` türünden değişkene değerini veriyor. Böylece `va_list` türünden değişkenin seçimlik ilk argümanı göstermesi sağlanıyor. Makronun ikinci parametresine işlevin türü belirtilerek isimlendirilmiş son parametresinin isminin geçilmesi gerekiyor.

## va_arg
```
va_arg (va_list arg, arg_type)
```
`va_arg` makrosu sıradaki seçimlik argümanın değerini döndürürken `va_list` türünden değişkenin de değerini değiştirerek onun bir sonraki seçimlik argümanı göstermesini sağlıyor. Makronun ikinci parametresine elde edilecek argümanın tür bilgisinin geçilmesi gerekiyor. Bu makroya döngüsel bir yapıda, seçimlik argümanların sayısı kadar çağrı yapılmasıyla işlevin `variadic` parametresine gönderilen tüm argümanlara erişilebiliyor.

## va_end makrosu

```
va_end (va_list ap)
```
Bu makro seçimlik argümanların dolaşılması sürecini sonlandırmak için çağrılıyor. `Variadic` işlevin çalışacak kodunun sonlanmasından önce bu makroyu çağırmak gerekiyor.

## va_copy makrosu

```
void va_copy(va_list dest, va_list source);
```

Bu makro dile `C99` standartları ile eklendi. `va_start` makrosu çağrılarak ilk değerini almış `va_list` türünden değişkenin bu makro ile kopyası çıkartabiliyor. Böylece argümanları birden fazla kez dolaşmak kolayca mümkün hale geliyor.

## variadic parametreye gönderilen argümanların kullanılması
İşlevlerin normal parametre değişkenlerini isimleri yoluyla kullanıyoruz. Ancak `variadic` parametreye ilişkin argümanların gönderildiği parametrelerin isimleri yok. Bu durumda işlev tanımında onları nasıl kullanacağız? İşleve gönderilen argümanlara yalnızca, standart `stdarg.h` başlık dosyasında tanımlanan özel makroları kullanarak, fonksiyon çağrısı ile gönderildikleri sıra ile erişilebiliyor. Standart `va_start`, `va_arg` ve `va_end` makrolarını kullanarak 3 aşamalı bir sürecin oluşturulması gerekiyor:

Önce `va_start` makrosunu kullanarak `va_list` türünden bir pointer değişkene değer verilmesi gerekiyor. Bu yapıldığıda `va_list` türünden değişken seçimlik ilk argümanı gösteriyor hale geliyor.
Daha sonra `va_arg` makrosuna döngüsel bir yapıda çağrı yaparak tüm seçimlik argümanlara erişilebiliyor. Yani `va_arg` makrosuna yapılan ilk çağrı ilk argümanı, ikinci çağrı ikinci argümanı veriyor. Argümanların hepsinin dolaşılması zorunlu değil. İşleve gönderilen argümanların bir kısmına erişmemek herhangi bir şekilde tanımsız davranış oluşturmuyor. Ancak gönderilen argüman sayısından daha fazla sayıda argümana erişim girişimi tanımsız davranış.

Argümanların elde edilmesi işleminin bitirildiğini ifade etmek için `va_list` türünden değişken ile `va_end` makrosuna çağrı yapılması gerekiyor. Aslında derleyicilerin çoğu `va_end` makrosu karşılığı hiçbir kod üretmiyor. Yine de hem standartlara tam olarak uygun bir kod oluşturmak için hem de kodun okunmasını kolaylaştırmak `va_end` makrosu mutlaka çağrılmalı.

## işlev tanımında seçimlik argümanların sayısının elde edilmesi
`variadic` işlevlerde argüman sayısının elde edilmesi için birden fazla teknik kullanılabilir.

İşlevin tam sayı türünden bir parametresi (tipik olarak birinci parametre) çağıran koddan işleve gönderilen diğer argümanların sayısını alır. Bu uygulanması en kolay tekniktir. Şimdi bu tekniği kullanan bir işlevi kodlayalım:

```
#include <stdarg.h>
#include <stdio.h>

int sum(int count, ...) 
{
	int sum = 0;
	va_list args;

	va_start(args, count);

	for (int i = 0; i < count; i++) {
		sum += va_arg(args, int);
	}

	va_end(args);

	return sum;
}

int main()
{
	int a = sum(2, 15, 20);
	int b = sum(3, 10, 15, 20);
	int c = sum(4, a, b, 10, 10);

	printf("c = %d\n", c);
}
```
Yukarıdaki kodda tanımlanan `sum` isimli `variadic` işlev kendisine gönderilen tam sayıların toplamını hesaplıyor. Aşağıdaki kodda yine aynı tekniği kullanan `min` ve `max` isimli işlevlerin tanımları yer alıyor. Bu işlevler kendilerine gönderilen tam sayılardan en küçük ve en büyük olanlarının değerlerini buluyorlar:

```
#include <stdarg.h>
#include <stdio.h>
#include <limits.h>

int min(int count, ...) 
{
	int min = INT_MAX;
	va_list args;
	va_start(args, count);

	for (int i = 0; i < count; i++) {
		int ival = va_arg(args, int);
		if (ival < min)
			min = ival;
	}
	va_end(args);

	return min;
}

int max(int count, ...)
{
	int max = INT_MIN;
	va_list args;
	va_start(args, count);

	for (int i = 0; i < count; i++) {
		int ival = va_arg(args, int);
		if (ival > max)
			max = ival;
	}
	va_end(args);

	return max;
}

int main()
{
	int x, y, z, t;

	printf("dort sayi giriniz: ");
	scanf("%d%d%d%d", &x, &y, &z, &t);

	printf("min = %d\n", min(4, x, y, z, t));
	printf("max = %d\n", max(4, x, y, z, t));
}
```
Bir başka teknik `variadic` işlevin kendisini çağıran koddan bir yazının adresini alarak kendisine gönderilen seçimlik argümanların sayısını bu yazıdan elde etmesi. `stdio` kütüphanesinde bildirilen standart `scanf` ve `printf` işlevleri de bu tekniği kullanıyor. Aşağıda `printf` işlevini sarmalayan basitleştirilmiş bir `print` işlevi tanımlıyoruz:

```
#include <stdio.h>
#include <stdarg.h>
#include <ctype.h>

void print(const char* pfmt, ...)
{
	va_list args;
	va_start(args, pfmt);

	while (*pfmt != '\0') {
		int ch = toupper(*pfmt);
		if (ch == 'D') {
			int i = va_arg(args, int);
			printf("%d\n", i);
		}
		else if (ch == 'C') {
			int c = va_arg(args, int);
			printf("%c\n", c);
		}
		else if (ch == 'F') {
			double d = va_arg(args, double);
			printf("%f\n", d);
		}
		else if (ch == 'U') {
			unsigned int uval = va_arg(args, unsigned int);
			printf("%u\n", uval);
		}
		++pfmt;
	}

	va_end(args);
}

int main(void)
{
	print("DcFfu", 3, 'a', 1.999, 42.5, 98u);
}
```
Çağrıyı yapan kod, işleve son argüman olarak başka bir argüman gönderilmeyeceğini belirten özel bir değer gönderebilir. Aşağıdaki kodu inceleyelim:

```
#include <stdarg.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>

char *concat(const char *ptr, ...)
{
	va_list args1, args2;

	va_start(args1, ptr);
	va_copy(args2, args1);
	const char *p;

	size_t len = strlen(ptr);
	while ((p = va_arg(args1, const char *)) != NULL)
		len += strlen(p);

	char *pd = (char *)malloc(len + 1);
	if (!pd) {
		fprintf(stderr, "bellek yetersiz\n");
		exit(EXIT_FAILURE);
	}
	va_end(args1);

	strcpy(pd, ptr);

	while ((p = va_arg(args2, const char *)) != NULL)
		strcat(pd, p);

	va_end(args2);

	return pd;
}

int main(void)
{
	char *p = concat("necati ", "ergin ", "kaan ", "aslan", NULL);

	puts(p);
	free(p);
}
```

Yukarıdaki kodda ismi `concat` olan istenilen sayıda argümanla çağrılabilen bir işlev tanımlanıyor. `concat` işlevi kendisine adresleri gönderilen yazıları elde ettiği dinamik bir bellek bloğunda birleştiriyor. İşlevi çağıran kod, birleştirilecek son yazının adresinden sonra işleve başka bir yazı gönderilemediğini bildirmek amacıyla işleve `NULL` gösterici geçiyor. Yukarıdaki kodda `va_copy` makrosunun kullanılması ile seçimlik argümanların iki kez dolaşıldığını görüyorsunuz.
