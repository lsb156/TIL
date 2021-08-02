

## 1\. Overview

<span style="color:  #333333;;">The symmetric-key block cipher plays an important role in data encryption. It means that the same key is used for both encryption and decryption. The [Advanced Encryption Standard](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) (AES) is a widely used symmetric-key encryption algorithm.</span>
<span style="color:  #333333;;">In this tutorial, we’ll see how to implement AES encryption and decryption using the Java Cryptography Architecture (JCA) within the JDK.</span>

<br>
## 2\. AES Algorithm
<br>
<span style="color:  #333333;;">The AES algorithm is an iterative, symmetric-key block cipher that **supports cryptographic keys (secret keys) of 128, 192, and 256 bits to encrypt and decrypt data in blocks of 128 bits**. The below figure shows the high-level AES algorithm:</span>

[![High Level AES Algorithm](https://www.baeldung.com/wp-content/uploads/2020/11/Figures.png)](https://www.baeldung.com/wp-content/uploads/2020/11/Figures.png)<span style="color:  #333333;;">If the data to be encrypted does not meet the block size of 128 bits requirement, it must be padded. Padding is a process of filling up the last block to 128 bits.</span>

## 3\. AES Variations
<br>
<span style="color:  #333333;;">The AES algorithm has six modes of operation:</span>

1. ECB (Electronic Code Book)
2. CBC (Cipher Block Chaining)
3. CFB (Cipher FeedBack)
4. OFB (Output FeedBack)
5. CTR (Counter)
6. GCM (Galois/Counter Mode)

<span style="color:  #333333;;">The mode of operation may be applied in order to strengthen the effect of the encryption algorithm. Moreover, the mode of operation may convert the block cipher into a stream cipher. Each mode has its strength and weakness. Let’s have a quick review.</span>

### 3.1. ECB

<span style="color:  #333333;;">This mode of operation is the simplest of all. The plaintext is divided into blocks with a size of 128 bits. Then each block will be encrypted with the same key and algorithm. Therefore, it produces the same result for the same block. This is the main weakness of this mode and **it is not recommended for encryption**. It requires padding data.</span>

### 3.2. CBC

<span style="color:  #333333;;">In order to overcome the ECB weakness, CBC mode uses an [Initialization Vector](https://en.wikipedia.org/wiki/Initialization_vector) (IV) to augment the encryption. First, CBC uses the plaintext block xor with the IV. Then it encrypts the result to the ciphertext block. In the next block, it uses the encryption result to xor with the plaintext block until the last block.</span>

<span style="color:  #333333;;">In this mode, encryption can not be parallelized, but decryption can be parallelized. It also requires padding data.</span>

### 3.3. CFB

<span style="color:  #333333;;">This mode can be used as a stream cipher. First, it encrypts the IV, then it will xor with the plaintext block to get ciphertext. Then CFB encrypts the encryption result to xor the plaintext. It needs an IV.</span>

<span style="color:  #333333;;">In this mode, decryption can be parallelized but encryption can not be parallelized.</span>

### 3.4. OFB

<span style="color:  #333333;;">This mode can be used as a stream cipher. First, it encrypts the IV. Then it uses the encryption results to xor the plaintext to get ciphertext.</span>

<span style="color:  #333333;;">It doesn’t require padding data and will not be affected by the noisy block.</span>

### 3.5. CTR

<span style="color:  #333333;;">This mode uses the value of a counter as an IV. It's very similar to OFB, but it uses the counter to be encrypted every time instead of the IV.</span>
<span style="color:  #333333;;">This mode has two strengths, including encryption/decryption parallelization, and noise in one block does not affect other blocks.</span>

### 3.6. GCM

<span style="color:  #333333;;">This mode is an extension of the CTR mode. The GCM has received significant attention and is recommended by NIST. The GCM model outputs ciphertext and an authentication tag. The main advantage of this mode, compared to other operation modes of the algorithm, is its efficiency.</span>
<span style="color:  #333333;;">In this tutorial, we'll use the *AES/CBC/PKCS5Padding* algorithm because it is widely used in many projects.</span>

### 3.7. Size of Data After Encryption

<span style="color:  #333333;;">As mentioned earlier, the AES has a block size of 128 bits or 16 bytes. The AES does not change the size, and the ciphertext size is equal to the cleartext size. Also, in ECB and CBC modes, we should use a padding algorithm likes *PKCS 5.* So, the size of data after encryption is:</span>

```
ciphertext_size (bytes) = cleartext_size + (16 - (cleartext_size % 16))
```

<span style="color:  #333333;;">For storing IV with ciphertext, we need to add 16 more bytes.</span>

## 4\. AES Parameters

<span style="color:  #333333;;">In the AES algorithm, we need three parameters: input data, secret key, and IV. IV is not used in ECB mode.</span>

### 4.1. Input Data

<span style="color:  #333333;;">The input data to the AES can be string, file, object, and password-based.</span>

### 4.2. Secret Key

<span style="color:  #333333;;">There are two ways for generating a secret key in the AES: generating from a random number or deriving from a given password.</span>
<span style="color:  #333333;;">**In the first approach, the secret key should be generated from a Cryptographically Secure (Pseudo-)Random Number Generator like the *SecureRandom* class.**</span>

<span style="color:  #333333;;">For generating a secret key, we can use the *KeyGenerator* class. Let’s define a method for generating the AES key with the size of *n* (128, 192, and 256) bits:</span>
<br>
```
public static SecretKey generateKey(int n) throws NoSuchAlgorithmException {
    KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
    keyGenerator.init(n);
    SecretKey key = keyGenerator.generateKey();
    return key;
}
```

<span style="color:  #333333;;">**In the second approach, the AES secret key can be derived from a given password using a password-based key derivation function like PBKDF2**. We also need a salt value for turning a password into a secret key. The salt is also a random value.</span>

<span style="color:  #333333;;">We can use the *SecretKeyFactory* class with the *PBKDF2WithHmacSHA256* algorithm for generating a key from a given password.</span>
<span style="color:  #333333;;">Let’s define a method for generating the AES key from a given password with 65,536 iterations and a key length of 256 bits:</span>
<br>
```
public static SecretKey getKeyFromPassword(String password, String salt)
    throws NoSuchAlgorithmException, InvalidKeySpecException {

    SecretKeyFactory factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA256");
    KeySpec spec = new PBEKeySpec(password.toCharArray(), salt.getBytes(), 65536, 256);
    SecretKey secret = new SecretKeySpec(factory.generateSecret(spec)
        .getEncoded(), "AES");
    return secret;
}
```

### 4.3. Initialization Vector (IV)

<span style="color:  #333333;;">IV is a pseudo-random value and has the same size as the block that is encrypted. We can use the *SecureRandom* class to generate a random IV.</span>
<span style="color:  #333333;;">Let’s define a method for generating an IV:</span>
<br>
```
public static IvParameterSpec generateIv() {
    byte[] iv = new byte[16];
    new SecureRandom().nextBytes(iv);
    return new IvParameterSpec(iv);
}
```

## 5\. Encryption and Decryption

5.1. String
<span style="color:  #333333;;">To implement input string encryption, we first need to generate the secret key and IV according to the previous section. As the next step, we create an instance from the *Cipher* class by using the *getInstance()* method.</span>
<span style="color:  #333333;;">Additionally, we configure a cipher instance using the *init()* method with a secret key, IV, and encryption mode. Finally, we encrypt the input string by invoking the *doFinal()* method. This method gets bytes of input and returns ciphertext in bytes:</span>
<br>
```
public static String encrypt(String algorithm, String input, SecretKey key,
    IvParameterSpec iv) throws NoSuchPaddingException, NoSuchAlgorithmException,
    InvalidAlgorithmParameterException, InvalidKeyException,
    BadPaddingException, IllegalBlockSizeException {

    Cipher cipher = Cipher.getInstance(algorithm);
    cipher.init(Cipher.ENCRYPT_MODE, key, iv);
    byte[] cipherText = cipher.doFinal(input.getBytes());
    return Base64.getEncoder()
        .encodeToString(cipherText);
}
```

<span style="color:  #333333;;">For decrypting an input string, we can initialize our cipher using the *DECRYPT\_MODE* to decrypt the content:</span>

```
public static String decrypt(String algorithm, String cipherText, SecretKey key,
    IvParameterSpec iv) throws NoSuchPaddingException, NoSuchAlgorithmException,
    InvalidAlgorithmParameterException, InvalidKeyException,
    BadPaddingException, IllegalBlockSizeException {

    Cipher cipher = Cipher.getInstance(algorithm);
    cipher.init(Cipher.DECRYPT_MODE, key, iv);
    byte[] plainText = cipher.doFinal(Base64.getDecoder()
        .decode(cipherText));
    return new String(plainText);
}
```

<span style="color:  #333333;;">Let's write a test method for encrypting and decrypting a string input:</span>

```
@Test
void givenString_whenEncrypt_thenSuccess()
    throws NoSuchAlgorithmException, IllegalBlockSizeException, InvalidKeyException,
    BadPaddingException, InvalidAlgorithmParameterException, NoSuchPaddingException { 

    String input = "baeldung";
    SecretKey key = AESUtil.generateKey(128);
    IvParameterSpec ivParameterSpec = AESUtil.generateIv();
    String algorithm = "AES/CBC/PKCS5Padding";
    String cipherText = AESUtil.encrypt(algorithm, input, key, ivParameterSpec);
    String plainText = AESUtil.decrypt(algorithm, cipherText, key, ivParameterSpec);
    Assertions.assertEquals(input, plainText);
}
```

### 5.2. File

<span style="color:  #333333;;">Now let's encrypt a file using the AES algorithm. The steps are the same, but we need some *IO* classes to work with the files. Let's encrypt a text file:</span>
<br>
```
public static void encryptFile(String algorithm, SecretKey key, IvParameterSpec iv,
    File inputFile, File outputFile) throws IOException, NoSuchPaddingException,
    NoSuchAlgorithmException, InvalidAlgorithmParameterException, InvalidKeyException,
    BadPaddingException, IllegalBlockSizeException {

    Cipher cipher = Cipher.getInstance(algorithm);
    cipher.init(Cipher.ENCRYPT_MODE, key, iv);
    FileInputStream inputStream = new FileInputStream(inputFile);
    FileOutputStream outputStream = new FileOutputStream(outputFile);
    byte[] buffer = new byte[64];
    int bytesRead;
    while ((bytesRead = inputStream.read(buffer)) != -1) {
        byte[] output = cipher.update(buffer, 0, bytesRead);
        if (output != null) {
            outputStream.write(output);
        }
    }
    byte[] outputBytes = cipher.doFinal();
    if (outputBytes != null) {
        outputStream.write(outputBytes);
    }
    inputStream.close();
    outputStream.close();
}
```

<span style="color:  #333333;;">Please note that trying to read the entire file – particularly if it is large – into memory is not recommended. Instead, we encrypt a buffer at a time.</span>
<span style="color:  #333333;;">For decrypting a file, we use similar steps and initialize our cipher using *DECRYPT\_MODE* as we saw before.</span>
<span style="color:  #333333;;">Again, let's define a test method for encrypting and decrypting a text file. In this method, we read the *baeldung.txt* file from the test resource directory, encrypt it into a file called *baeldung.encrypted*, and then decrypt the file into a new file:</span>
<br>
```
@Test
void givenFile_whenEncrypt_thenSuccess() 
    throws NoSuchAlgorithmException, IOException, IllegalBlockSizeException, 
    InvalidKeyException, BadPaddingException, InvalidAlgorithmParameterException, 
    NoSuchPaddingException {

    SecretKey key = AESUtil.generateKey(128);
    String algorithm = "AES/CBC/PKCS5Padding";
    IvParameterSpec ivParameterSpec = AESUtil.generateIv();
    Resource resource = new ClassPathResource("inputFile/baeldung.txt");
    File inputFile = resource.getFile();
    File encryptedFile = new File("classpath:baeldung.encrypted");
    File decryptedFile = new File("document.decrypted");
    AESUtil.encryptFile(algorithm, key, ivParameterSpec, inputFile, encryptedFile);
    AESUtil.decryptFile(
      algorithm, key, ivParameterSpec, encryptedFile, decryptedFile);
    assertThat(inputFile).hasSameTextualContentAs(decryptedFile);
}
```

### 5.3. Password-Based

<span style="color:  #333333;;">We can do the AES encryption and decryption using the secret key that is derived from a given password.</span>
<span style="color:  #333333;;">For generating a secret key, we use the *getKeyFromPassword()* method. The encryption and decryption steps are the same as those shown in the string input section. We can then use the instantiated cipher and the provided secret key to perform the encryption.</span>
<span style="color:  #333333;;">Let's write a test method:</span>
<br>
```
@Test
void givenPassword_whenEncrypt_thenSuccess() 
    throws InvalidKeySpecException, NoSuchAlgorithmException, 
    IllegalBlockSizeException, InvalidKeyException, BadPaddingException, 
    InvalidAlgorithmParameterException, NoSuchPaddingException {

    String plainText = "www.baeldung.com";
    String password = "baeldung";
    String salt = "12345678";
    IvParameterSpec ivParameterSpec = AESUtil.generateIv();
    SecretKey key = AESUtil.getKeyFromPassword(password,salt);
    String cipherText = AESUtil.encryptPasswordBased(plainText, key, ivParameterSpec);
    String decryptedCipherText = AESUtil.decryptPasswordBased(
      cipherText, key, ivParameterSpec);
    Assertions.assertEquals(plainText, decryptedCipherText);
}
```

### 5.4. Object

<span style="color:  #333333;;">For encrypting a Java object, we need to use the *SealedObject* class. The object should be *Serializable*. Let's begin by defining a *Student* class:</span>
<br>
```
public class Student implements Serializable {
    private String name;
    private int age;

    // standard setters and getters
}
```

<span style="color:  #333333;;">Next, let's encrypt the *Student* object :</span>
<br>
```
public static SealedObject encryptObject(String algorithm, Serializable object,
    SecretKey key, IvParameterSpec iv) throws NoSuchPaddingException,
    NoSuchAlgorithmException, InvalidAlgorithmParameterException, 
    InvalidKeyException, IOException, IllegalBlockSizeException {

    Cipher cipher = Cipher.getInstance(algorithm);
    cipher.init(Cipher.ENCRYPT_MODE, key, iv);
    SealedObject sealedObject = new SealedObject(object, cipher);
    return sealedObject;
}
```

<span style="color:  #333333;;">The encrypted object can later be decrypted using the correct cipher:</span>
<br>
```
public static Serializable decryptObject(String algorithm, SealedObject sealedObject,
    SecretKey key, IvParameterSpec iv) throws NoSuchPaddingException,
    NoSuchAlgorithmException, InvalidAlgorithmParameterException, InvalidKeyException,
    ClassNotFoundException, BadPaddingException, IllegalBlockSizeException,
    IOException {

    Cipher cipher = Cipher.getInstance(algorithm);
    cipher.init(Cipher.DECRYPT_MODE, key, iv);
    Serializable unsealObject = (Serializable) sealedObject.getObject(cipher);
    return unsealObject;
}
```

<span style="color:  #333333;;">Let's write a test case:</span>
<br>
```
@Test
void givenObject_whenEncrypt_thenSuccess() 
    throws NoSuchAlgorithmException, IllegalBlockSizeException, InvalidKeyException,
    InvalidAlgorithmParameterException, NoSuchPaddingException, IOException, 
    BadPaddingException, ClassNotFoundException {

    Student student = new Student("Baeldung", 20);
    SecretKey key = AESUtil.generateKey(128);
    IvParameterSpec ivParameterSpec = AESUtil.generateIv();
    String algorithm = "AES/CBC/PKCS5Padding";
    SealedObject sealedObject = AESUtil.encryptObject(
      algorithm, student, key, ivParameterSpec);
    Student object = (Student) AESUtil.decryptObject(
      algorithm, sealedObject, key, ivParameterSpec);
    assertThat(student).isEqualToComparingFieldByField(object);
}
```

## 6\. Conclusion

<span style="color:  #333333;;">In summary, we've learned how to encrypt and decrypt input data like strings, files, objects, and password-based data, using the AES algorithm in Java. Additionally, we've discussed the AES variations and the size of data after encryption.</span>
<span style="color:  #333333;;">As always, the full source code of the article is available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-security-2).</span>


> 출처 :  [https://www.baeldung.com/java-aes-encryption-decryption](https://www.baeldung.com/java-aes-encryption-decryption)
