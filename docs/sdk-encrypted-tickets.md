# Authorizing GOG GALAXY Users in Third-Party Services

The GOG GALAXY SDK provides access to Encrypted App Tickets, which can be used by a game for seamless GOG user authorization in any third-party backend.

Encrypted App Tickets are created and encrypted in the GOG GALAXY backend using a shared Private Key. The game can request the ticket for the current user, and the ticket can be passed to any third-party backend that also knows the Private Key and will be able to decrypt the data and thus confirm the user’s identity and their license for the game. The whole process is shown on the following picture:

![Authentication in a 3rd-Party Backend Using GOG Service](_assets/encrypted-app-tickets.png)

## Retrieving the Encrypted App Ticket

1. Request the ticket using the [`IUser::RequestEncryptedAppTicket()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IUser.html#a29f307e31066fc39b93802e363ea2064) method of the GOG GALAXY SDK and provide any additional elements in its `data` parameter. By default, the following data is enclosed in the ticket:
    - `user_id`

    - `client_id`

    - timestamp of ticket generation.

2. The response to this call comes to [`IEncryptedAppTicketListener`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IEncryptedAppTicketListener.html).
3. If the request was successful ([`OnEncryptedAppTicketRetrieveSuccess()`](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IEncryptedAppTicketListener.html#ae5ad1ca3940e9df9ebc21e25799aa084)), you can retrieve the ticket with the [`IUser::GetEncryptedAppTicket() `](https://docs.gog.com/galaxyapi/classgalaxy_1_1api_1_1IUser.html#a96af6792efc260e75daebedca2cf74c6) method.

!!! Important
    encrypted_ticket.private_key and encryptedAppTicket are already encoded using base64url.

## Using the Ticket in a Third-Party Backend

App Tickets are encrypted using [AES (Rijndael)](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) with a 128-bit block size, 256-bit encryption key in the CBC mode.

In order to use the ticket, first you will need to transfer `encrypted_ticket.private_key` for your game to your backend service:

1. Go to the [GOG Developer Portal](https://devportal.gog.com).

2. Click *Games* in the main menu.

3. In the resulting list of your games, click the *SDK Credentials* button for your desired game:

    ![SDK Credentials](_assets/games-sdk-credentials.png)

4. A pop-up window appears with the SDK Credentials. Copy the value of the third entry (`encrypted_ticket.private_key`):

    ![Encrypted Ticket Private Key](_assets/sdk-encrypted-ticket-key.png)

5. Use this value for the `CLIENT_PRIVATE_KEY` variable in either of the scripts.

!!! Important
    Be advised, ticket and private_key are incomplete. Both are missing necessary padding(=) at the end of each.

The following scripts are sample implementations of decoding EncryptedAppTickets in a third-party backend.

=== "PHP"

    In our [GitHub repository](https://github.com/gogcom/galaxy-session-tickets-php) you will find a simple PHP library which decodes encrypted tickets. The script below makes use of this library in a PHP-based backend.
    
    ``` php
    <?php
    
    require_once('vendor/autoload.php');
    
    $mcryptPlaintext = \GOG\SessionTickets\MCryptSessionTicketDecoder::decode(
        \GOG\SessionTickets\ExampleData::ENCRYPTED_SESSION_TICKET,
        \GOG\SessionTickets\ExampleData::CLIENT_PRIVATE_KEY
    );
    
    $openSSLplaintext = \GOG\SessionTickets\OpenSSLSessionTicketDecoder::decode(
        \GOG\SessionTickets\ExampleData::ENCRYPTED_SESSION_TICKET,
        \GOG\SessionTickets\ExampleData::CLIENT_PRIVATE_KEY
    );
    
    if ($openSSLplaintext === $mcryptPlaintext && $mcryptPlaintext === \GOG\SessionTickets\ExampleData::PLAINTEXT_SESSION_TICKET) {
        echo $openSSLplaintext."\n";
    } else {
        throw new \Exception('Encrypted Session Ticket decryption failed');
    }
    ```

=== "Python"

This python example shows how to decode both ticket and private as both are base64url encoded.
First make sure if padding is correct as both ticket and the key might be missing `=` symbols.
Then using decoded private key we use AES decryption to extract our ticket.

    ``` python
    import base64
    from Crypto.Cipher import AES
    
    def decrypt(base64_encrypted_ticket, base64_secret):
        encrypted_ticket = base64.urlsafe_b64decode(base64_encrypted_ticket)
        secret = base64.urlsafe_b64decode(base64_secret)
        ivSize = AES.block_size
        cipher = AES.new(secret, AES.MODE_CBC, encrypted_ticket[:ivSize])
        data =  cipher.decrypt(encrypted_ticket[ivSize:])
        return data
        
    ticket = ''
    key = ''
    
    # restore missing padding truncated on galaxy backend(?)
    ticket += '=' * (-len(ticket) % 4)
    key += '=' * (-len(key) % 4)
    
    decrytped_data = decrypt(ticket, key)
    
    print(base64.urlsafe_b64encode(decrytped_data))
    print(decrytped_data)
    ```

=== "C#"

CSharp example is analogous to the Python one except here we first convert base64url encoded ticket and key to base64.
``` C#
using System;
using System.Collections;
using System.IO;
using System.Security.Cryptography;
using System.Text;

class Program
{
    static byte[] decrypt(string ticket, string key)
    {
        // convert base64
        byte[] decodedTicketBytes = Convert.FromBase64String(ticket);
        byte[] decodedSecretBytes = Convert.FromBase64String(key);

        // IV is the first 16 bytes of the ticket
        byte[] iv = decodedTicketBytes[0..16];

        // encrypted data is the rest of the ticket
        byte[] bytesWithoutIV = decodedTicketBytes[16..^0];

        using (Aes aes = Aes.Create())
        {
            aes.Key = decodedSecretBytes;
            aes.IV = iv;
            aes.Padding = PaddingMode.Zeros;

            byte[] decryptedBytes = null;
            using (ICryptoTransform decryptor = aes.CreateDecryptor())
            {
                decryptedBytes = decryptor.TransformFinalBlock(bytesWithoutIV, 0, bytesWithoutIV.Length);
            }

            return decryptedBytes;
        }
    }
    static void Main()
    {

        string ticket = "";
        string key = "";

        // Convert base64url to base64
        ticket = ticket.Replace("-", "+").Replace("_", "/");
        key = key.Replace("-", "+").Replace("_", "/");

        // Fix missing padding
        ticket = ticket.PadRight(ticket.Length + (4 - ticket.Length % 4) % 4, '=');
        key = key.PadRight(key.Length + (4 - key.Length % 4) % 4, '=');

        byte[] decryptedData = decrypt(ticket, key);

        Console.WriteLine(Convert.ToBase64String(decryptedData));
        Console.WriteLine(System.Text.Encoding.UTF8.GetString(decryptedData));
    }
}
``` 
