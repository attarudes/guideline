暗号化
================================================================================

.. only:: html

 .. contents:: 目次
    :local:

Overview
--------------------------------------------------------------------------------
| Spring Securityの主機能は「認証」、「認可」であるが、暗号化に関するユーティリティも含まれている。
| ただし、提供される機能は限定的なものであるため、Spring Securityが対応していない暗号化処理については、別途実装する必要がある。
| 本ガイドラインでは、Spring Securityを利用した共通鍵暗復号 / 疑似乱数の生成処理、および、Java (JCA)での公開鍵暗復号処理について説明する。

| Spring Securityにおける暗号化機能の詳細については\ `公式リファレンス <http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/reference/htmlsingle/#crypto>`_\ を参照されたい。

暗号化方式
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 暗号化方式について説明する。

共通鍵暗号化方式
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 暗号化、および、復号の際に同じ鍵を利用する方式である。
| 暗 / 復号処理のコストは低いが、復号側であらかじめ鍵を保持しておく必要があり、鍵を安全に受け渡すことが難しいというデメリットがある。

公開鍵暗号化方式
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 公開されている鍵(公開鍵)で暗号化し、非公開の鍵(秘密鍵)で復号する方式である。
| 鍵の受け渡しが不要であるためセキュリティ的には優れるが、暗 / 復号処理のコストは高い。

ハイブリッド方式
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| 共通鍵暗号化方式と公開鍵暗号化方式を組み合わせた方法であり、SSLなどで利用されている。
| たとえば、HTTPS(サーバー認証)では、共通鍵を公開鍵で暗号化したうえで送信し、受信側(サーバー側)で秘密鍵を利用して復号する。
| 以降は、その共通鍵を利用して暗号化通信を行う、という仕組みである。

暗号化アルゴリズム
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 暗号化アルゴリズムについて説明する。

DES / 3DES
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| DES (Data Encryption Standard)
| 3DES (トリプルDES)

DES (Data Encryption Standard) は共通暗号化方式のアルゴリズムとして、アメリカ合衆国の標準規格として規格化されたものである。鍵長が56ビットと短いことから、現在では推奨されていない。3DES、トリプルDESは、鍵を変えながらDESを繰り返す暗号化アルゴリズムである。

AES
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| AES (Advanced Encryption Standard)は共通鍵暗号化方式のアルゴリズムである。DESの後継として制定された暗号化規格であり、暗号化における現在のデファクトスタンダードとして利用されている。

RSA
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| RSAは公開鍵暗号化方式のアルゴリズムである。素因数分解の困難性に基づいているため、計算機の能力向上により危殆化することとなる。いわゆる「暗号化アルゴリズムの2010年問題」として指摘されているように充分な長さが必要であり、現時点では2048ビットが標準的に利用されている。

DSA / ECDSA
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| DSA (Digital Signature Algorithm) は、デジタル署名のための標準規格である。離散対数問題の困難性に基づいている。ECDSA (Elliptic Curve DSA : 楕円曲線DSA)は、楕円曲線暗号を用いたDSAの変種である。楕円曲線暗号においては、セキュリティレベルを確保するために必要となる鍵長が短くなるというメリットがある。

疑似乱数 (生成器)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| 暗号においては、鍵の生成などで乱数が用いられる。
| このとき、乱数として生成される値が予測可能であると、暗号の安全性が保てなくなるため、結果の予測が困難な乱数(疑似乱数)を利用する必要がある。
| 疑似乱数の生成に用いられるのが疑似乱数生成器である。

Cipher
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| CipherはAESやRSAなどの暗号化アルゴリズム、ECBやCBCなどの暗号モード、PKCS1などのパディング形式の組み合わせである。
| Javaアプリケーションでは、\ ``"RSA/ECB/PKCS1Padding"``\ という形で指定する。

Spring Securityにおける暗号化機能
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Spring Securityでは、共通鍵暗号化方式での暗 / 復号機能を提供している。
| 暗号化アルゴリズムは256-bit AES using PKCS #5’s PBKDF2 (Password-Based Key Derivation Function #2)である。

| 共通鍵暗号化方式であるため、鍵を受け渡す必要がない状況における暗号化、あるいは、公開鍵暗号化方式と組み合わせた形で利用に限定することが望ましい。

| Spring Securityでは、共通鍵暗号化方式での暗 / 復号機能として、以下のインターフェイスが用意されている。

* \ ``org.springframework.security.crypto.encrypt.TextEncryptor``\ 
* \ ``org.springframework.security.crypto.encrypt.BytesEncryptor``\ 

| これらの実装クラスとして、テキスト用の\ ``org.springframework.security.crypto.encrypt.HexEncodingTextEncryptor``\ クラスやバイト配列用の\ ``org.springframework.security.crypto.encrypt.AesBytesEncryptor``\ クラスが提供されている。

| \ ``TextEncryptor``\ / \ ``BytesEncryptor``\ の仕組みとして、\ ``encrypt``\ メソッドで暗号化を行ない、\ ``decrypt(byte[] encryptedBytes)``\ メソッドで復号処理を行なう。
  
| また、乱数(鍵)生成の機能として、以下のインターフェイスが用意されている。

* \ ``org.springframework.security.crypto.keygen.StringKeyGenerator``\ 
* \ ``org.springframework.security.crypto.keygen.BytesKeyGenerator``\ 

| 前者の実装クラスとして、テキスト用の \ ``org.springframework.security.crypto.keygen.HexEncodingStringKeyGenerator``\ クラスが提供されており、後者の実装クラスとして、バイト配列用の以下のクラスが提供されている。

* \ ``org.springframework.security.crypto.keygen.SecureRandomBytesKeyGenerator``\ 
* \ ``org.springframework.security.crypto.keygen.SharedKeyGenerator``\ 

| \ ``StringKeyGenerator``\ / \ ``BytesKeyGenerator``\ の仕組みとして、\ ``generateKey``\ メソッドで乱数(鍵)の生成を行なう。

How to use
--------------------------------------------------------------------------------

| JavaでAESの鍵長256ビットを扱うためには、強度が無制限のJCE管轄ポリシーファイルを適用する必要がある。

      .. note::
         輸入規制の関係上、Javaではデフォルトの暗号化アルゴリズム強度が制限されている。より強力なアルゴリズムを利用する場合は、強度が無制限のJCE管轄ポリシーファイルを入手し、JDK/JREにインストールする必要がある。詳細については、\ `Java 暗号化アーキテクチャー Oracle プロバイダのドキュメント <http://docs.oracle.com/javase/jp/7/technotes/guides/security/SunProviders.html#importlimits>`_\を参照されたい。

         JCE管轄ポリシーファイルのダウンロード先
	 
         * \ `Java 8 用 <http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html>`_\
         * \ `Java 7 用 <http://www.oracle.com/technetwork/java/embedded/embedded-se/downloads/jce-7-download-432124.html>`_\

TextEncryptorによるテキストの暗号化
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

  .. code-block:: java

    public static String encryptText(
        String secret, String salt, String rawText) {
        TextEncryptor encryptor = Encryptors.text(secret, salt); // (1)

        return encryptor.encrypt(rawText); // (2)
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 共通鍵とソルトを指定して\ ``TextEncryptor``\ オブジェクトのインスタンスを生成する。
         | このときに指定した共通鍵、ソルトは、復号時にも同じものを利用することになる。

     * - | (2)
       - | \ ``encrypt``\ メソッドで暗号化処理を実行する。

同一の暗号化結果を得る
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

  .. code-block:: java

    public static void encryptTextResult(
        String secret, String salt, String rawText) {
        TextEncryptor encryptor1 = Encryptors.text(secret, salt); // (1)
        TextEncryptor encryptor2 = Encryptors.queryableText(secret, salt); // (2)

        System.out.println(encryptor1.encrypt(rawText)); // (3)
        System.out.println(encryptor1.encrypt(rawText)); // 
        System.out.println(encryptor2.encrypt(rawText)); // (4)
        System.out.println(encryptor2.encrypt(rawText)); //
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | \ ``text``\ メソッドで生成される\ ``TextEncryptor``\ オブジェクトは、初期化ベクトルがランダムであるため、暗号化の際に異なる結果を返す。

     * - | (2)
       - | 暗号化した結果として同じ値が必要な場合は、\ ``queryableText``\ メソッドを利用してインスタンスを生成する。

     * - | (3)
       - | \ ``text``\ メソッドで生成した\ ``TextEncryptor``\ オブジェクトは、\ ``encrypt``\ メソッドでの暗号化の結果として異なる値を返す。ただし、当然のことながら、鍵とソルトが同一であれば、暗号化の結果が異なっている場合でも、復号処理の結果は同一になる(正しく復号できる)。
	 
     * - | (4)
       - | \ ``queryableText``\ メソッドで生成した\ ``TextEncryptor``\ オブジェクトは、\ ``encrypt``\ メソッドでの暗号化の結果として同一の値を返す。暗号化した結果を用いてデータベースの検索を行なうようなケースで利用できる。

         | セキュリティ強度が落ちる点を踏まえ、使用の可否を検討してほしい。

TextEncryptorによるテキストの復号
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

  .. code-block:: java

    public static String decryptText(String secret, String salt, String encryptedText) {
        TextEncryptor decryptor = Encryptors.text(secret, salt); // (1)

        return decryptor.decrypt(encryptedText); // (2)
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 共通鍵とソルトを指定して\ ``TextEncryptor``\ オブジェクトのインスタンスを生成する。
         | 共通鍵、ソルトは、暗号化した際に利用したものを指定する。

     * - | (2)
       - | 暗号化されたテキストを\ ``decrypt``\ メソッドで復号する。

BytesEncryptorによるバイト配列の暗号化
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

  .. code-block:: java

    public static byte[] encryptBytes(String secret, String salt, byte[] rawBytes) {
        BytesEncryptor encryptor = Encryptors.standard(secret, salt); // (1)

        return encryptor.encrypt(rawBytes); // (2)
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 共通鍵とソルトを指定して\ ``BytesEncryptor``\ オブジェクトのインスタンスを生成する。
         | このときに指定した共通鍵、ソルトは、復号時にも同じものを利用することになる。

     * - | (2)
       - | \ ``encrypt``\ メソッドで暗号化処理を実行する。

BytesEncryptorによるバイト配列の復号
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

  .. code-block:: java

    public static byte[] decryptBytes(String secret, String salt, byte[] encryptedBytes) {
        BytesEncryptor decryptor = Encryptors.standard(secret, salt);

        return decryptor.decrypt(encryptedBytes); // (1)
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 共通鍵とソルトを指定して\ ``BytesEncryptor``\ オブジェクトのインスタンスを生成する。
         | 共通鍵、ソルトは、暗号化した際に利用したものを指定する。

     * - | (2)
       - | \ ``decrypt``\ メソッドで復号処理を実行する。

ByteKeyGeneratorによるバイト配列型の疑似乱数 / 鍵生成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

  .. code-block:: java

    public static void createDifferentBytesKey() {
        BytesKeyGenerator generator = KeyGenerators.secureRandom(); // (1)
        System.out.println(Arrays.toString(generator.generateKey())); // (2)
        System.out.println(Arrays.toString(generator.generateKey())); //
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 鍵(疑似乱数)生成器\ ``BytesKeyGenerator``\ オブジェクトのインスタンスを生成する。
         | この生成器で鍵を生成すると、毎回異なる値が生成される。
         |
         | 鍵長を指定しない場合、デフォルトで8バイトの鍵が生成される。

     * - | (2)
       - | \ ``generateKey``\ メソッドで鍵(疑似乱数)を生成する。

  .. code-block:: java

    public static void createSameBytesKey() {
        BytesKeyGenerator generator = KeyGenerators.shared(16); // (1)
        System.out.println(Arrays.toString(generator.generateKey())); // (2)
        System.out.println(Arrays.toString(generator.generateKey())); //
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 鍵生成器\ ``BytesKeyGenerator``\ オブジェクトのインスタンスを生成する。
         | この生成器で鍵を生成すると、毎回同じ値が生成される。
         |
         | 鍵長の指定は必須である。

     * - | (2)
       - | \ ``generateKey``\ メソッドで鍵を生成する。

StringKeyGeneratorによる String 型の疑似乱数生成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

  .. code-block:: java

    public static void createStringKey() {
        StringKeyGenerator generator = KeyGenerators.string(); // (1)
        System.out.println(generator.generateKey()); // (2)
        System.out.println(generator.generateKey()); //
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 鍵 (疑似乱数) 生成器\ ``BytesKeyGenerator``\ オブジェクトのインスタンスを生成する。
         | この生成器で鍵を生成すると、毎回異なる値が生成される。
         |
         | 鍵長は指定できず、常に8バイトの鍵が生成される。

     * - | (2)
       - | \ ``generateKey``\ メソッドで鍵を生成する。

Appendix
--------------------------------------------------------------------------------

| Spring Securityでは公開鍵暗号化方式に関する機能は提供されていないため、Java(JCA)、および、OpenSSL を利用した方法について説明する。

JCAでキーペアを生成し、JCAで暗 / 復号
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| JCA (Java Cryptography Architecture)でキーペア(公開鍵 / 秘密鍵の組み合わせ)を生成し、公開鍵で暗号化、秘密鍵で復号処理を行う。

JCAによるキーペアの生成
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

  .. code-block:: java

    public void generateKeysByJCA() {
        try {
            KeyPairGenerator generator = KeyPairGenerator.getInstance("RSA"); // (1)
            generator.initialize(2048); // (2)
            KeyPair keyPair = generator.generateKeyPair(); // (3)
            PublicKey publicKey = keyPair.getPublic();
            PrivateKey privateKey = keyPair.getPrivate();

            byte[] encryptedBytes = encryptByPublicKey("Hello World!", publicKey);  // (4)
            System.out.println(Base64.getEncoder().encodeToString(encryptedBytes));
            String decryptedText = decryptByPrivateKey(encryptedBytes, privateKey); // (5)
            System.out.println(decryptedText);
        } catch (NoSuchAlgorithmException ignored) {
            throw new IllegalStateException("Should not be happend!", ignored);
        }
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | RSAアルゴリズムを指定して\ ``KeyPairGenerator``\ オブジェクトのインスタンスを生成する。

     * - | (2)
       - | 鍵長として2048ビットを指定する。

     * - | (3)
       - | キーペアを生成する。

     * - | (4)
       - | 公開鍵を利用して暗号化処理を行なう。処理内容は後述する。

     * - | (5)
       - | 秘密鍵を利用して復号処理を行なう。処理内容は後述する。
	 

公開鍵による暗号化
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 公開鍵を利用して文字列を暗号化する。

  .. code-block:: java

    public byte[] encryptByPublicKey(String rawText, PublicKey publicKey) {
        try {
            Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding"); // (1)
            cipher.init(Cipher.ENCRYPT_MODE, publicKey);                       // (2)
            return cipher.doFinal(rawText.getBytes(Charset.forName("UTF-8"))); //
        } catch (NoSuchAlgorithmException | NoSuchPaddingException ignored) {
            throw new IllegalStateException("Should not be happened!", ignored);
        } catch (InvalidKeyException |
                 IllegalBlockSizeException |
                 BadPaddingException e) {
            throw new IllegalArgumentException(e);
        }
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 暗号化アルゴリズム、暗号モード、パディング方式を指定して、\ ``Cipher``\ オブジェクトのインスタンスを生成する。

     * - | (2)
       - | 暗号化処理を実行する。

秘密鍵による復号
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| 秘密鍵を利用してバイト配列を復号する。

  .. code-block:: java

    public String decryptByPrivateKey(byte[] encryptedBytes, PrivateKey privateKey) {
        try {
            Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding"); // (1)
            cipher.init(Cipher.DECRYPT_MODE, privateKey);           // (2)
            byte[] decryptedBytes = cipher.doFinal(encryptedBytes); //
            return new String(decryptedBytes, Charset.forName("UTF-8"));
        } catch (NoSuchAlgorithmException | NoSuchPaddingException ignored) {
            throw new IllegalStateException("Should not be happened!", ignored);
        } catch (InvalidKeyException |
                 IllegalBlockSizeException |
                 BadPaddingException e) {
            throw new IllegalArgumentException(e);
        }
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 暗号化アルゴリズム、モード、パディング方式を指定して、\ ``Cipher``\ オブジェクトのインスタンスを生成する。

     * - | (2)
       - | 復号処理を実行する。

OpenSSLで作成したキーペアを利用してJCAで暗号化、OpenSSLで復号
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Cipherが同一であれば、公開鍵暗号化方式は別の方法で暗 / 復号を行なうことが可能である。
| ここでは、OpenSSLを利用してあらかじめキーペアを作成しておき、その公開鍵を利用してJCAによる暗号化を行なう。また、その秘密鍵を利用してOpenSSLで復号処理を行なう方法を説明する。

| 事前準備として、以下の処理を行う。

  .. code-block:: console

     $ openssl genrsa -out private.pem 2048  # (1)

     $ openssl pkcs8 -topk8 -nocrypt -in private.pem -out private.pk8 -outform DER  # (2)

     $ openssl rsa -pubout -in private.pem -out public.der -outform DER  # (3)

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | OpenSSLで2048ビットの秘密鍵(DER形式)を生成する。

     * - | (2)
       - | Javaアプリケーションから読み込むために、秘密鍵をPKCS #8形式に変換する。

     * - | (3)
       - | 秘密鍵から公開鍵(DER形式)を生成する。

JCAによる暗号化、OpenSSLによる復号
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| OpenSSLを利用してキーペアを作成済みであるため、アプリケーションでは公開鍵の読み込み、および、その公開鍵を利用した暗号化処理を行なう。

  .. code-block:: java

    public void useOpenSSLDecryption() {
        try {
            KeySpec publicKeySpec = new X509EncodedKeySpec(
                    Files.readAllBytes(Paths.get("public.der"))); // (1)
            KeyFactory keyFactory = KeyFactory.getInstance("RSA");
            PublicKey publicKey = keyFactory.generatePublic(publicKeySpec); // (2)

            byte[] encryptedBytes = encryptByPublicKey("Hello World!", publicKey); // (3)

            Files.write(Paths.get("encryptedByJCA.txt"), encryptedBytes);
            System.out.println("Please execute the following command:");
            System.out
                    .println("openssl rsautl -decrypt -inkey hoge.pem -in encryptedByJCA.txt");
        } catch (IOException e) {
            throw new UncheckedIOException(e);
        } catch (NoSuchAlgorithmException ignored) {
            throw new IllegalStateException("Should not be happend!", ignored);
        } catch (InvalidKeySpecException e) {
            throw new IllegalArgumentException(e);
        }
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 公開鍵ファイルからバイナリデータを読み込む。

     * - | (2)
       - | バイナリデータから\ ``PublicKey``\ オブジェクトを生成する。

     * - | (3)
       - | 公開鍵を利用して暗号化処理を行なう。


| プログラム実行後に以下の処理を実行する。

  .. code-block:: console

     $ openssl rsautl -decrypt -inkey private.pem -in encryptedByJCA.txt  # (1)

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | 秘密鍵を利用してOpenSSLで復号する。


OpenSSLで作成したキーペアを利用してOpenSSLで暗号化、JCAで復号
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| 事前準備として、以下の処理を行う。

  .. code-block:: console

     $ openssl genrsa -out private.pem 2048  # (1)

     $ openssl pkcs8 -topk8 -nocrypt -in private.pem -out private.pk8 -outform DER  # (2)

     $ openssl rsa -pubout -in private.pem -out public.der -outform DER  # (3)

     $ echo Hello | openssl rsautl -encrypt -keyform DER -pubin -inkey public.der -out encryptedByOpenSSL.txt  # (4)
     
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | OpenSSLで2048ビットの秘密鍵(DER形式)を生成する。

     * - | (2)
       - | Javaアプリケーションから読み込むために、秘密鍵をPKCS #8形式に変換する。
     * - | (3)
       - | 秘密鍵から公開鍵(DER形式)を生成する。

     * - | (4)
       - | 公開鍵を利用してOpenSSLで暗号化する。

OpenSSLによる暗号化、JCAによる復号
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| OpenSSLを利用してキーペアを作成済みであるため、アプリケーションでは秘密鍵の読み込み、および、その秘密鍵を利用した復号処理を行なう。

  .. code-block:: java

    public void useOpenSSLEncryption() {
        try {
            KeySpec privateKeySpec = new PKCS8EncodedKeySpec(
                    Files.readAllBytes(Paths.get("private.pk8"))); // (1)
            KeyFactory keyFactory = KeyFactory.getInstance("RSA");
            PrivateKey privateKey = keyFactory.generatePrivate(privateKeySpec); // (2)

            String decryptedText = decryptByPrivateKey(
                   Files.readAllBytes(Paths.get("encryptedByOpenSSL.txt")),
                   privateKey); // (3)
            System.out.println(decryptedText);
        } catch (IOException e) {
            throw new UncheckedIOException(e);
        } catch (NoSuchAlgorithmException ignored) {
            throw new IllegalStateException("Should not be happend!", ignored);
        } catch (InvalidKeySpecException e) {
            throw new IllegalArgumentException(e);
        }
    }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - 項番
       - 説明
     * - | (1)
       - | PKCS #8形式の秘密鍵ファイルからバイナリデータを読み込み\ ``PKCS8EncodedKeySpec``\ オブジェクトのインスタンスを生成する。

     * - | (2)
       - | \ ``KeyFactory``\ オブジェクトから\ ``PrivateKey``\ オブジェクトを生成する。

     * - | (3)
       - | 秘密鍵を利用して復号処理を行なう。


.. raw:: latex

   \newpage

