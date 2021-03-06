# Hasura GraphQL Engine ile geliştirilmiş bir API Servisini Vue.js ile Kullanmak

API'ler için türlendirilmiş _(typed)_ sorgulama dillerinden birisi olarak öne çıkan GraphQL'e bir süredir uğramıyordum. Daha doğrusu GraphQL sorgusu çalıştırılabilecek şekilde API servis hazırlıklarını yapmaya üşeniyordum. Bu nedenle işi kolaylaştıran ve Heroku üzerinden sunulan Hasura GraphQL Engine hizmetine bakmaya karar verdim. Hasura, veriyi PostgreSQL kullanarak saklıyor ve ayrıca API'yi bir Docker Container içerisinden sunuyor. Amacım Hasura tarafında hazırlayacağım iki kobay veri setini, Vue.js tabanlı bir istemcisinden tüketmek. Basitçe listeleme yapıp, veri ekleme işini yapabilsem başlangıç aşamasında yeterli olur. İşe Hasura servisini hazırlayarak başlamak lazım.

## Hasura GraphQL Engine Tarafının Geliştirilmesi

Pek tabii Heroku üzerinde bir hesabımızın olması gerekiyor. Sonrasında [şu](https://elements.heroku.com/) adrese gidip elements kısmından Hasura GraphQL Engine'i seçmek yeterli. 

![assets/credit_1.png](assets/credit_1.png)

Gelinen yerden _Deploy to Heroku_ diyerek projeyi oluşturabiliriz.

![assets/credit_2.png](assets/credit_2.png)

Ben aşağıdaki bilgileri kullanarak bir proje oluşturdum.

![assets/credit_3.png](assets/credit_3.png)

Deploy başarılı bir şekilde tamamlandıktan sonra, 

![assets/credit_4.png](assets/credit_4.png)

_View_ seçeneği ile yönetim paneline geçebiliriz.

![assets/credit_5.png](assets/credit_5.png) 

Dikkat edileceği üzere GraphQL sorgularını çalıştırabileceğimiz hazır bir arayüz sunuluyor. Ancak öncesinde örnek veri setleri hazırlamalıyız. Bunun için Data sekmesinden yararlanabiliriz.

![assets/credit_6.png](assets/credit_6.png)

Arabirimin kullanımı oldukça kolay. Ben aşağıdaki özelliklere sahip tabloları oluşturdum.

![assets/credit_7.png](assets/credit_7.png)

categories isimli tablomuzda unique tipte, get_random_uuid() fonksiyonu ile eklenen satır için rastgele üretilen categoryId ve text tipinde title isimli alanlar bulunuyor. categoryId, aynı zamanda primary key türünden bir alan.

![assets/credit_8.png](assets/credit_8.png)

products tablosunda da UUID tipinde productId, text tipinde description, number tipinde listPrice ve yine UUID tipinde categoryId isimli alanlar mevcut. categoryId alanını, ürünleri kategoriye bağlamak için _(foreign key relations)_ kullanıyoruz. Ama bu alanı foreign key yapmak için _Modify_ pencersine geçmeliyiz.

![assets/credit_9.png](assets/credit_9.png)

![assets/credit_10.png](assets/credit_10.png)

İlişkinin geçerlilik kazanması içinse, categories tablosunun _Relationships_ penceresine gidip önerilen bağlantıyı eklemek gerekir.

![assets/credit_14.png](assets/credit_14.png)

![assets/credit_15.png](assets/credit_15.png)

![assets/credit_16.png](assets/credit_16.png)

>Bu durumda categories üzerinden products'a gidebiliriz. Ters ilişkiyi de kurabiliriz ve bir ürünle birlikte bağlı olduğu kategorinin bilgisini de yansıtabiliriz ki ürünleri çektiğimizde hangi kategoride olduğunu da göstermek güzel olur. Bunu nasıl yapabileceğinizi bir deneyin isterim.

Hasura'nın Postgresql tarafındaki örnek tablolarımız hazır. İstersek _Insert Row_ pencersinden tablolara örnek veri girişleri yapabilir ve GraphiQL pencresinden sorgular çalıştırabiliriz. Ben yaptığım denemelerle alakalı bir kaç örnek ekran görüntüsü paylaşayım. Arabirimin sağ tarafında yer alan Docs menüsüne de bakabilirsiniz. Burada query ve mutation örnekleri, hazırladığımız veri setleri için otomatik olarak oluşturuluyorlar.

![assets/credit_11.png](assets/credit_11.png)

![assets/credit_12.png](assets/credit_12.png)

### Örnek Sorgular

İşte arabirim üzerinden denediğim bir kaç örnek sorgu.

Kategorilerin başlıkları,

```
query{
  categories{
    title
  }
}
```

![assets/credit_19.png](assets/credit_19.png)

Kategorilere bağlı ürünler

```
query{
  categories{
    title
    products{
      description
      listPrice
    }
  }
}
```

![assets/credit_18.png](assets/credit_18.png)

Ürünlerin tam listesi ve bağlı olduğu kategori adları

```
query{
  products{
    description
    listPrice
    category{
      title
    }
  }
}
```

![assets/credit_17.png](assets/credit_17.png)

Veri girişi de yapabiliriz. Bunun için mutation kullanıldığını daha önceden öğrenmiştim. Örneğin yeni bir kategoriyi aşağıdaki gibi ekleyebiliriz.

```
mutation {
  insert_categories(objects: [{
    title: "Çorap",
  }]) {
    returning {
      categoryId
    }
  }
```
![assets/credit_20.png](assets/credit_20.png)

Hasura, GraphQL API’si arkasında PostgreSQL veri tabanını kullanırken SQLden aşina olduğumuz bir çok sorgulama metodunu da hazır olarak sunar. Örneğin fiyatı 300 birimin üstünde olan ürünleri aşağıdaki sorgu ile çekebiliriz.

```
{
  products(where: {listPrice: {_gt: 300}}) {
    description
    listPrice
    category {
      title
    }
  }
}
```

![assets/credit_21.png](assets/credit_21.png)

Where metodu sorgu şemasına otomatik olarak eklenmiştir. _gt tahmin edileceği üzere greater than anlamındadır. Yukarıdaki sorguya fiyata göre tersten sıralama seçeneği de koyabiliriz. Sadece where koşulu arkasından order_by çağrısı yapmamız yeterlidir.

```
{
  products(where: {listPrice: {_gt: 300}}, order_by: {listPrice: desc}) {
    description
    listPrice
    category {
      title
    }
  }
}
```

![assets/credit_22.png](assets/credit_22.png)

Çok büyük veri setleri düşünüldüğünde önyüzler için sayfalama önemlidir. Bunun için limit ve offset değerlerini kullanabiliriz. 5nci üründen itibaren 5 ürünün getirilmesi için aşağıdaki sorgu kullanılabilir.

```
{
  products(limit: 5, offset: 5) {
    description
    listPrice
    category {
      title
    }
  }
}
```

![assets/credit_23.png](assets/credit_23.png)

Hasura Query Engine’in sorgu seçenekleri ile ilgili olarak [buradaki dokümanı](https://docs.hasura.io/1.0/graphql/manual/queries/index.html) takip edebiliriz.

## İstemci _(Vue)_ Tarafı

İstemci tarafını basit bir Vue uygulaması olarak geliştirmeye karar vermiştim. Aşağıdaki terminal komutunu kullanıp varsayılan ayarları ile projeyi oluşturdum. Ayrıca GraphQL tarafı ile konuşabilmek için gerekli npm paketlerini de yükledim. Apollo _(ilerleyen ünitelerde ondan bir GraphQL Server yazmayı deneyeceğim)_, GraphQL servisimiz ile kolay bir şekilde iletişim kurmamızı sağlayacak.

>Görsel taraf için bootstrap kullanmayı tercih ettim.

```
sudo vue create nba-client
sudo npm install vue-apollo apollo-client apollo-cache-inmemory apollo-link-http graphql-tag graphql bootstrap --save
```

## Yapılanlar

Vue uygulaması tarafından yaptıklarım kısaca şöyle _(Kod dosyalarındaki yorum bloklarında daha detaylı bilgiler mevcut)_

- Components klasörüne tek ürün için kullanılabilecek Product isimli bir bileşen eklendi _(anasayfa listelemesinde tekrarlanacak türden bir bileşen)_
- Main.js içeriği Hasura'yı ve GraphQL'i kullanabilir hale getirildi
- Ürünlerin listesini gösterebilmek için components klasörüne ProductList bileşeni eklendi
- Yeni bir ürün ekleyebilmek için ProductAdd isimli bileşen geliştirildi
- App.vue içeriği eklenen bileşenleri kullanacak şekilde düzenlendi

## TODO _(Benim tembelliğimden size düşen)_

><span style="color:red">Bu servisi JWT Authentication bünyesine almak lazım. İşte size güzel bir araştırma konusu. Başlangıç noktası olarak [Auto0'ın şu dokümanına bakılabilir](https://auth0.com/docs/quickstart/spa/vuejs). Ben şu an için sadece HASURA_GRAPHQL_ADMIN_SECRET kullanarak servis adresine erişimi kısıtladım. </span>

## Çalışma Zamanı

Uygulamayı çalıştırmak için,

```
npm run serve
```

terminal komutunu kullanabiliriz. Sonrasında http://localhost:8080 adresine gitmemiz yeterli.

![assets/credit_24.png](assets/credit_24.png)

Yeni ürün ekleme bileşeni konulduktan sonrasına ait örnek bir ekran görüntüsü.

![assets/credit_25.png](assets/credit_25.png)

Hatta yeni bir forma eklediğimizde gönderilen Graphql Mutation sorgusundan dönen değeri de F12 sonrası Network sekmesinden yakalayabiliriz.

![assets/credit_26.png](assets/credit_26.png)

>throw new ToBeContinuedException("Yeni ürün ekleme sayfasında kategori seçiminde combobox kullanımı yapılmalı");

## Neler Öğrendim

- Heroku'da Docker Container içerisinde çalışan ve PostgreSQL verileri GraphQL ile sorgulanabilir olarak sunan Hasura isimli bir motor olduğunu
- Hasura arabirimden ilişkili tabloları nasıl oluşturabileceğimi
- Bir kaç basit GraphQL sorgusunu _(sorgularda sayfalama yapmak, ilişkili veri getirmek, where koşulları kullanmak)_
- Vue tarafında GraphQL sorgularının nasıl gönderilebileceği
- Component içinde Component kullanımlarını
- Temel component tasarlama adımlarını
- Vue tarafından bir Mutation sorgusunun nasıl gönderilebileceğini ve schema veri tiplerine dikkat etmem gerektiğini