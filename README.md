# maysoft-di
Dependency Injection

SOLID diye kısaltılan prensiplerden en sonunda ki harf tarafından temsil edilen Dependency Injection programcılara fazlasıyla yardımcı olan bir prensip. Kendisini açıklamadan önce daha kalıcı olması ve daha kolay anlaşılması için kendisini oluşturan kelimelerin anlaşılmasında yarar var. Dependency bağımlı olunan şey demek. Mesela, bir aşçının yemeğini yapabilmesi için kullanmak zorunda olduğu ve dolayısıyla bağımlı olduğu araçlardan bir tanesinin bıçak olması gibi. Kod yazarken de çoğu zaman hazır yazılmış başka sınıfları kendi kodunuz içinde kullandığınızdan, kullandığınız bu diğer sınıflara da dependency denir.
Injection ise günlük hayatta kullanılan manasına benzer şekilde kullanılmaktadır bu prensipte. Yani nasıl ki hasta olan bir insana dışarıdan ilacı enjekte ederek verirsiniz, aynı şekilde bir sınıfın bağımlı olduğu diğer sınıfları da bu sınıf içerisine dışarıdan enjekte edebilirsiniz. Dolayısıyla iki kelimeyi bir araya getirdiğinizde bu prensip ile ulaşmaya çalıştığınız amaç ortaya çıkmaktadır. Yani bağımlı olduğunuz sınıfları, kendi sınıflarınız içine enjekte ederek kullanmak.
Şimdiye kadar anlattığımız kısımların daha rahat anlaşılması için bir örnek verelim ve sonrasında ise neden DI (Dependency Injection) kullanmak istersiniz, ne gibi faydalarını görebilirsiniz gibi soruları cevaplamaya çalışalım.

public class AccountCreator {

  private AccountChecker _accountChecker;
  private AccountRepository _accountRepository;
  
  public AccountCreator(){
    _accountChecker = new AccountChecker();
    _accountRepository = new AccountRepository();
  }
  
  public bool TryCreateAccount(AccountInfo accountInfo){
    if(!_accountChecker.Exists(accountInfo.AccountNumber){
      _accountCreator.Create(accountInfo);
      return true;
    }
    return false;
  }
}

Yukarıda ki gibi bir sınıfımızın olduğunu düşünelim. Burada ki mantık şu: AccountCreator isminde, yeni bir hesap oluşturulmasını sağlayan bir sınıfımız var. Bu sınıfımız başka iki sınıfa bağımlı. Bağımlı olduğu bu sınıflar da, yeni bir hesap oluşturan AccountRepositor, ve oluşturulmak istenilen hesap daha öceden oluşturulmuş mu diye bakanAccountChecker. Şimdi bu kodu ileride değiştirmek istersek karşımıza çıkacak sorunları bir inceleyim.
Neden var olan kodu değiştirmek istemeyiz?
Var olan bir kodu ne kadar değiştirseniz, projenizde sorun çıkarma riskiniz de o denli artacaktır.
DI meselesini daha iyi anlamak için öncelikle incelenmesi gereken bir sorun var. Bir koda bakım yapıyorsanız, o zaman yapmak isteyeceğiniz en son şeylerden bir tanesidir var olan kodu değiştirmek. Çünkü var olan kod uzun zaman boyunca test edilmiş ve sorunları devamlı düzeltilmiş olup yapacağınız bazı masum değişikler bile o kısımların yeniden test edilmesini ve oraya bağımlı olan kodların da değişmek zorunda kalmasıyla sonuçlanabilir. Bunun içindir ki SOLID prensiplerinden bir tanesi de Open/Closed Principle (OCP) olarak geçmektedir. OCP kısaca bir kod değişime kapalı ama genişletmeye açık olmalıdır demektir. Yani bir koda yeni şeyler ekleyebilirsiniz, mesela yeni sınıflar gibi. Ama var olan sınıfları ve sınıflar içinde ki methodların içeriğini değiştirmeniz, yeni sorunların çıkmasına neden olabileceğinden tavsiye edilmez. Bir de çalışan ve çalıştığı uzun zaman boyunca production sunucularında kullanılarak ispat edilmiş kısımlara dokunmak demek değiştirdiğiniz kodunuzun aynı güvenirliğe ulaşması için yine uzun bir zaman boyunca kullanılması demek olabilir. Kısacası var olan koda ancak çok kapsayıcı şekilde yazılmış Unit Testleriniz varsa dokunmanızı tavsiye ederim. Aksi halde yeni sorunların çıkma ihtimali yükselir. Bu şekilde kapsayıcı unit testler yazmak için de Dependency Injection hayati bir önem arzediyor. Kısacası DI ile hem kapsayıcı unit testler yazabilecek hem de kodunuzu değiştirmek yerine extend edebileceksiniz. Bir taşla iki kuş…
Yukarıda önemimi vurgulayarak değindiğim mantığı anlatmamda ki sebep ilk örnekte önemli bir şeyin eksik olması. Dikkat ederseniz AccountRepository ve AccountChecker sınıfları bir interface aracılığı ile gelmiyor. Bilakis, kendilerine bağımlı olan sınıfın içinde oluşturuluyorlar. İleride yeni bir eklenti yapmak istediğimizde, yukarıda anlatılan sorunlar ile karşılacağız. Neden? Çünkü yapmak zorunda olduğumuz değişikler önceden yazılmış kodların değişmesiyle sonuçlanacak. Adım adım sorunu ve çözümü incelemeye başlayalım.
Diyelim ki AccountChecker sınıfı veri tabanına bakarak bir hesabın var olup olmadığına karar veriyor. Ama bazı müşterilerimiz veri tabanı yerine XML dosyalarını kullanıyor olabilirler. O zaman AccountChecker sınıfını ya modifiye edeceğiz ya da AccountCheckerXml isminde yeni sınıf oluşturarak gerekli algoritmaları bu sınıf içinde yazacağız. Ama bu iki yaklaşım da şuan ki tasarımda sorunlar meydana getiriyor. Birinci durumda, AccountChecker sınıfının içi değişiyor. Dolayısıyla tüm sistem yeniden düzgün bir şekilde test edilmesi gerekiyor. Eğer AccountCheckerXml diye yeni sınıf eklemek istemez ve yeni methodlarımızı AccountChecker sınıfı içine koymak istersek, o zaman da bu sınıfın ileride alacağı hali bir düşünün. Bir zaman sonra hesap bilgilerini Amazon Web servislerinde tutan bir müşteri ile çalışmamız gerekirse? Derken bu sınıf belki de onlarca farklı implementasyonu içeren ve giderek büyüyen bir sınıf haline gelecek. Dolasıyla yukarıda ki basitliğinden ve okunabilirliğinden zamanla eser kalmayacak. Bir örnek üzerinden gidelim:

public class AccountChecker{
  public bool Exists(Account account){
    //.... 
  }
  
  private string ExtractName(Account account){
    // ...
  }
}

Bir kod yazıldığından daha fazla okunur.
Yukarıda ki örnek anlaşılır kodlar içeriyor. Kısa methodlar, amacını doğru bir şekilde açıklayan method ve değişken isimleri ve basit ve kısa bir şekilde yazılmış bir sınıf. Bir kod yazıldığından daha fazla okunur. Dolayısıyla rahat okunması için yaptığınız küçük dokunuşlar bile uzun zaman arkanızdan güzel şeyler söylenmesine vesile olacaktır. Aynı şekilde yukarıda ki kod sadece bir amaç uğruna çalışmaktadır. Şimdi aynı sınıf içine var olan methodların bir de XML ile çalışan versiyonunu, sonra Amazon web servis, sonra Azure vs. diye devamlı yeni methodlar eklersiniz, bu okunması kolay kod zamanla devasa bir sınıfa dönüşecek ve büyüklüğüne doğru orantılı olarak okunması da zorlaşmaya başlayacak:

public class AccountChecker{
  public bool Exists(Account account){
    //.... 
  }
   
  public bool ExistsInAmazonWebServices(Account account){
    //.... 
  }
   public bool ExistsInAzure(Account account){
    //.... 
  }
   public bool ExistsInLDAP(Account account){
    //.... 
  }
   public bool ExistsInHeaven(Account account){
    //.... 
  }
  
  private string ExtractName(Account account){
    // ...
  }
  // OMG: This is getting crazy...
}
Peki çözüm? En basit çözüm kalıtım kullanmak olurdu. Aşağıda ki kodu incelemek istediğimizde bunu daha rahat bir şekilde göreceğiz:
public interface IAccountChecker
{
  bool Exists(AccountNumber accountNumber);
}

public class DatabaseAccountChecker: IAccountChecker {}

public class AzureAccountChecker: IAccountChecker {}

public class XmlAccountChecker: IAccountChecker {}

Yukarıda ki sınıfları ve interface’i ayrı dosyalar içerisine koyabilirsiniz. Bu problemi çözmek için alternatif çözümlerden birisi olan Strategy Pattern da kullanılabilinir ama şimdilik en kolay olanından devam edelim. Yukarıda ki kod içine örnek olması açısından sadece genel kısımlarını koydum. Ama dikkat ederseniz şimdi istediğim kadar sınıf oluşturabilirim. Ama şimdi update etmem gereken bir yer daha var: AccountCreator sınıfı:

public class AccountCreator {

  private IAccountChecker _accountChecker;
  private AccountRepository _accountRepository;
  
  public AccountCreator(){
    // We are still tigtly coupled to DatabaseAccountCreator concrete type.
    _accountChecker = new DatabaseAccountChecker();
    _accountRepository = new AccountRepository();
  }
  
  public bool TryCreateAccount(AccountInfo accountInfo){
    if(!_accountChecker.Exists(accountInfo.AccountNumber){
      _accountCreator.Create(accountInfo);
      return true;
    }
    return false;
  }
}

Belki AccountChecker sorununa bir nebze olsun bir çözüm bulmuş olduk ama ilk başta açıkladığım sorun hala burada devam ediyor, çünkü şimdi de AccountCreator sınıfının içini değiştirmek zorunda kaldım. AccountChecker sınıfına bağlı olan yüzlerce sınıf olduğunda, değiştirmek zorunda kalacağım yerleri bir düşündüğünüz zaman meselenin ciddiyeti daha bir anlaşılır olmaya başlıyor.
Biz yine sorunu AccountCreator üzerinden çözmeye çalışalım. Şimdi istiyorum ki, yeni bir AccountChecker sınıfı eklediğimde, AccountCreator sınıfını hiç değiştirmek zorunda kalmayayım. Dolayısıyla AccountCreator içinde, interface dışında AccountCheckergibi hiç bir sınıfa bağımlı olmamam lazım. Demek ki kullanmak istediğim AccountChecker sınıfını bir şekilde AccountCreator içine enjekte etmem gerekiyor. Bunun için constructor methodu gayet mantıklı duruyor. Çünkü bu sınıf verilmeden AccountCreator kullanılamaz durumda olacağından, bir tane account checker sınıfının enjekte edilmesini zorunlu hale getirmem gerekiyor:

public class AccountCreator {

  private IAccountChecker _accountChecker;
  private AccountRepository _accountRepository;
  
  public AccountCreator(IAccountChecker accountChecker){
    // We are still tigtly coupled to DatabaseAccountCreator concrete type.
    _accountChecker = accountChecker;
    _accountRepository = new AccountRepository();
  }
  
  public bool TryCreateAccount(AccountInfo accountInfo){
    if(!_accountChecker.Exists(accountInfo.AccountNumber){
      _accountCreator.Create(accountInfo);
      return true;
    }
    return false;
  }
}

Artık AccountCreator sınıfı içinde IAccountChecker interface’ini kalıtım alan bir hiç bir sınıfa bağlı değiliz. IAccountChecker interface’ini implement eden istedigim tipi constructor methoda inject etmek sureti ile kullanabilirim. Bu sayede AccountCreator sınıfı hiç bir değişikliğe uğramıyor. Bu uzun vadede bakımı çok daha kolay ve genişletilmesi çok daha rahat bir mimari oluşturmak demektir. Hatta DI kullandığınızda, merkezi bir mapping sayesinde, hangi interface’in hangi sınıf ile eşleşeceğini söylemek sureti ile, istediğiniz sınıfı istediğiniz constructor methoduna enjekte etmeniz çok daha kolaylaşmış oluyor. Bununla alakalı basit bir örneği Ninject DI framework’una değindiğim ilerideki kısımda gösterdim. Bu sayede, AccountChecker sınıfı yerine DatabaseAccountChecker sınıfını kullanmak istediğinizde, yüzlerce sınıfı değiştirmek yerine, bu global mapping kısmında yapacağınız değişiklik tüm sistemde etkili olacaktır.
Kendiniz basit bir injection yapmak istediğimizde bu aşağıda ki gibi gözükecektir:

public class Program{
  public static void Main(string[] args){
    AccountCreator accountCreator = new AccountCreator(new DatabaseAccountChecker());
    var accountInfo = GetAccountInfoFromSomewhere();
    var result = accountCreator.TryCreateAccount(accountInfo);
    Console.WriteLine("Account Creation was succesfull? " + result); 
  }
}

Yukarıda ki örnekte DatabaseAccountChecker sınıfını nasıl enjekte ettiğimiz görülmekte. Ama tabiki kurumsal projelerde mesele bu kadar basit olmayacağından, Dependency Injection olayını daha da basitleştirmek ve streamline hale getirmek için özelleşmiş framework’lar mevcut. Siz kendinizde bir framework yazabilirsiniz ama zaten yazılmışlar varken, neden Amerika’yı yeniden keşfetmek isteyeceksiniz ki (Bazı özel durumlar hariç.). Bu framework’lar kodunuzda kilit yerlere entegre edilmek suretiyle yukarıda ki örnekte ki gibi bağımlı olunan sınıfları elinizle teker teker enjekte etmeniz yerine otomatik bir şekilde sizin adınıza bu kısımları hallediyorlar. Siz sadece hangi interface ile hangi sınıfı birbirine bağlamak istediğinizi bu framework’lere söylüyorsuz. Mesela Ninject bu framework’lardan bir tanesi. Aşağıda ki kodda nasıl bağlama yaptığınızı gösteren küçük bir örnek:

public class WarriorModule : NinjectModule
{
    public override void Load() 
    {
        this.Bind<IWeapon>().To<Sword>();
    }
}

IWeapon interface’ini Sword alt tipine bağlıyor. Bu şu demek, Ninject ne zaman bir sınıfın IWeapon tipine bağımlı olduğunu görse, ona otomatik olarak Sword alt tipini oluşturup gönderecek. Tabi ki bunu farkı şekillerde customize edebilirsiniz. Mesela, Ninject’e eğer IWeapon tipi şu ve şu namespace’lerde bulunan tipler tarafından isteniyorsa, o zaman Sword yerine Knife alt tipini oluştur ve gönder de diyebilirsiniz. Hatta hangi tipin nasıl bağlanacağını bir XML ayar dosyasında bile tutabilir ve program çalışırken bunları rahatlıkla farklı müşteriler için ya da farklı amaçlar için değiştirebilirsiniz.
Ninject gibi daha bir çok DI framework var. Bunlara aynı zamanda IoC Container’lar da deniyor. Google ettiğiniz zaman bu isimlerde ararsanız karşınıza farklı özellikler sunan farklı framework’lar gelecetir. IoC kısaltmasının da ne demek olduğunu başka bir yazımda anlatacağım.

Sonuç
DI’nın belkide en büyük faydalarından bir tanesi de istediğiniz sınıfı rahatlıkla mock ederek kodunuzu Unit Test’ler yazarak rahatça test edebilmenizdir.
Şimdiye kadar DI’nin nasıl kullanılabileceğini basit bir örnek üzerinden anlattım. Konuyu kapatmadan önce DI’nin faydalarını kısaca bir özetleyelim:
Projede somut sınıflar üzerine olan bağımlılığı azaltacaktır ve dolayısıyla genel olarak bakımın daha kolay olmasını sağlayacak.
Modüller arası daha düzgün tasarlanmış dependency tree’ye sahip olacaksınız.
Programda istenilen sınıfların rahatlıkla değiştirilmesine yardımcı olacak. Çünkü bağımlılık olmadığı için bir sınıf yerine başka bir sınıf kullanmak istediğinizde değiştirmek zorunda olacağınız kod miktarı ya sıfır yada çok daha az olacak.
Uygulama çalışma anında sizlere daha rahat configuration yapma şansı tanıyacak. Mesela, XML kullanarak hangi interface ile hangi sınıfın birbirlerine bağlı olacağını kolayca tanımlayabilirsiniz. Bu farklı müşterilen farklı isteklerini kodunuzu değiştirmek zorunda kalmadan rahatlıkla sunabilmeniz demektir.
Hangi sınıfları kullanacağınızı bir yerden kontrol etmiş olacaksınız.
Belkide en önemlisi çok daha rahat Unit Test ler yazmanızı sağlamış olacağı. İstediğiniz sınıfı rahatlıkla mock ederek farklı kod kısımlarını test edebileceksiniz. Bunun hakkında da farklı bir blog post yazmayı planlıyorum.

Sonuç Kod
Kodun en son halini görmek isterseniz aşağıda görebilirsiniz. Çok bir şey değişmemiş gibi durabilir ama aslında mimarinin gelişmesi açısından dünyalar değişikler olmuş oldu:
public class AccountCreator{
  // Interface'ler tanımlıyoruz. Dolayısıyla kendi sınıflarımızı rahatlıkla kullanabiliriz.
  private IAccountChecker _accountChecker;
  private IAccountRepository _accountRepository;
  
  // Dependency'lerimizi constructor method vasıtasıyla enjekte ediyoruz.
  public AccountCreator(IAccountChecker accountChecker, IAccountRepository accountRepository){
    _accountChecker = accountChecker;
    _accountRepository = new accountRepository;
  }
  
  public void CreateAccount(AccountInfo accountInfo){
    if(HasAccountNumber(accountInfo)){
      throw new InvalidAccountInfo("New accounts cannot have account numbers");
    }
    if(_accountChecker.Exists(SanitizeUserName(accountInfo.UserName)){
      throw new UsernameExistsException("This username is already taken. Please use a different username");
    }
    _accountRepository.Create(GetAccountDataTransferObject(accountInfo));
  }
  
  private string SanitizeUserName(string username){
    var sanitizedUsername = username.Trim().ToLower().HtmlEncode();
  }
  
  private bool HasAccountNumber(AccountInfo accountInfo){
    return accountInfo.AccountNumber != null;
  }
  
  private AccountDTO GetAccountDataTransferObject(AccountInfo accountInfo){
    return new AccountDTO {
      FirstName = accountInfo.FirstName,
      LastName = accountInfo.LastName,
      UserName = accountInfo.UserName,
      AccountCreated = accountInfo.AccountCreated
    }
  }
}
