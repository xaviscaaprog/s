# Ücretsiz Oracle Cloud VPS kurulumu

Oracle Cloud, ücretli deneme kredisinden ayrı olarak bazı "Always Free" bilgisayar örnekleri sunuyor. Güncel seçenekler ve limitler [Oracle Free Tier](https://www.oracle.com/cloud/free/) sayfasında listeleniyor. Ücretsiz kaynakların bölgeye göre kapasitesi değişebilir ve hesap açarken ödeme kartı doğrulaması istenebilir. Herhangi bir ücretli kaynağı seçmeyin veya hesabı ücretli plana yükseltmeyin; ücretsiz kaynakların devamlılığı ve erişilebilirliği garanti değildir.

Oracle Cloud hesabı açıp Compute bölümünden bir örnek oluşturun. Görsel olarak Ubuntu 24.04 LTS, makine tipi olarak Ampere A1 Flex seçin ve örneğin oluşturma ekranında bu kaynağın "Always Free" uygunluğunu doğrulayın. Bot için 1 OCPU ve 6 GB RAM yeterlidir. Always Free etiketi görünmüyorsa devam etmeyin; o örnek ücret çıkarabilir. Oracle'ın ücretsiz kaynak ve koşulları değişebileceğinden oluşturma öncesi [resmi limitleri](https://www.oracle.com/cloud/free/) tekrar kontrol edin.

Bu servis botu VPS açıldığında başlatır ve kapanırsa yeniden çalıştırır. VPS çalışırken kendi PC'niz kapalı olabilir. Sunucu hesabını açma ve örneği oluşturma adımları hesap sahibinin kendisi tarafından yapılmalıdır.

## Dosyaları sunucuya aktarma

WinSCP ile VPS'e SFTP bağlantısı kurun. Sunucudaki kullanıcı dizininizde `cctr-discord-bot-upload` klasörü açın ve `discord-bot` klasörünün içeriğini buraya aktarın. `bot.db` dosyasını aktarın; mevcut oda, XP, ses süresi ve ekonomi verileriniz bu dosyadadır. `.env` dosyasını aktarmayın. `.env.example` aktarılabilir. `__pycache__` klasörü gerekli değildir.

## Kurulum

SSH ile sunucuya bağlanıp aşağıdaki komutları çalıştırın. `LINUX_KULLANICI_ADI` kısmını SSH ile bağlandığınız Linux kullanıcı adıyla değiştirin.

```bash
sudo apt update
sudo apt install -y python3 python3-venv python3-dev build-essential libffi-dev libsodium-dev
sudo adduser --system --group --home /opt/cctr-discord-bot discordbot
sudo cp -a /home/LINUX_KULLANICI_ADI/cctr-discord-bot-upload/. /opt/cctr-discord-bot/
sudo chown -R discordbot:discordbot /opt/cctr-discord-bot
sudo -u discordbot python3 -m venv /opt/cctr-discord-bot/.venv
sudo -u discordbot /opt/cctr-discord-bot/.venv/bin/pip install -r /opt/cctr-discord-bot/requirements.txt
```

Token dosyasını oluşturup düzenleyin:

```bash
sudo install -m 600 /dev/null /etc/cctr-discord-bot.env
sudo nano /etc/cctr-discord-bot.env
```

Dosyanın içine Discord bot tokeninizi şu biçimde girin; tırnak kullanmayın ve tokeni kimseyle paylaşmayın:

```text
DISCORD_TOKEN=DISCORD_BOT_TOKENINIZ
```

Servisi kurup başlatın:

```bash
sudo cp /opt/cctr-discord-bot/deploy/cctr-discord-bot.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now cctr-discord-bot
sudo systemctl status cctr-discord-bot
```

Çalışma kayıtlarını görmek için:

```bash
sudo journalctl -u cctr-discord-bot -f
```

Kod güncellemesinde Python kodu ve gerekli yapılandırma dosyalarını aktarın; sunucudaki
`/opt/cctr-discord-bot/bot.db` dosyasının üzerine yerel `bot.db` dosyasını kopyalamayın.
Ses süreleri, XP, ekonomi ve oda verileri bu veritabanında kalır. İsterseniz güncelleme
öncesi veritabanını yedekleyin. Ardından sahipliği düzeltip servisi yeniden başlatın:

```bash
sudo chown -R discordbot:discordbot /opt/cctr-discord-bot
sudo systemctl restart cctr-discord-bot
```

VPS sağlayıcısının ağ kuralları SSH için 22 numaralı porta izin vermelidir. Botun Discord'a bağlanması için VPS'ten dışarı HTTPS erişimi de açık olmalıdır.

## Sunucu olay kayıtları

Bot, `config.py` içindeki `AUDIT_LOG_CHANNEL_ID` kanalına üye katılma/ayrılma, takma ad
ve rol değişiklikleri, ses durumu, kanal/konu oluşturma-silme ve ayar değişiklikleri,
davet oluşturma-silme, sunucu ayarı değişiklikleri, mesaj düzenleme/silme ve
yasaklama, zaman aşımı, kick, webhook ve bot ekleme olaylarını gönderir. Eşleşen Discord
denetim kaydı varsa işlemi yapan ve gerekçe de eklenir. Kanal ve rol izin farkları,
davetin kullanım/süre bilgisi ve katılımda kullanılabilen davet kodu da kaydedilir.
Moderatör/gerekçe için botta **Denetim Kaydını Görüntüle**, davet kullanımını izlemek
için **Sunucuyu Yönet** izni olmalıdır. Hedef kanalda mesaj gönderme ve embed ekleme
izinleri de gerekir. Hedef kanalda mesaj gönderme ve embed ekleme izinleri de gerekir.
Mesaj içerikleri, Nitro bilgileri, rozetler veya özel hesap verileri kaydedilmez.

## Kız rolü başvuru ticket'ı

Sunucuda yönetici yetkisi olan bir hesap, panelin gönderileceği kanalda
`/kiz_rol_al_paneli` komutunu çalıştırıp verilecek rolü ve başvuruları inceleyecek
yetkili rolünü seçer; komut paneli aynı kanala gönderir. Ticket kanallarının taşınacağı
kategori ve sesli görüşme kanalı isteğe bağlıdır. Üyeler paneldeki **Kız rolü almak
istiyorum** butonuyla başvuru ticket'ı açar. Komut ayarları botun `bot.db`
veritabanında saklanır. Panel ve ticket
butonları bot yeniden başlatıldığında çalışmaya devam eder.

Üye panelden ticket açınca yalnızca kendisi, yetkili rolü ve botun görebildiği özel bir
kanal oluşturulur. Yetkili üye ile sesli görüşmeyi yaptıktan sonra **Sesli doğrulandı —
rolü ver** butonuna basarak seçilen rolü verebilir. Başvuran veya yetkili ekip
**Ticket'ı kapat** butonuyla ticket'ı silebilir. Botta **Kanalları Yönet** ve
**Rolleri Yönet** izinleri olmalı; atanacak rol botun en üst rolünün altında
bulunmalıdır. Sesli görüşme bot tarafından kaydedilmez.
