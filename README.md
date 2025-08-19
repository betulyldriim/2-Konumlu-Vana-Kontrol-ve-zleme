Bu proje, bir PLC (Siemens S7-319-3 PN/DP) kullanılarak, iki konumlu (açık/kapalı) bir vananın kontrolü ve durum izlemesi için tasarlanmıştır. Projenin ana mantığı, modüler bir yaklaşımla, özel bir fonksiyon bloğu (valve_2Pos) içinde yer almaktadır. Bu, aynı vana kontrol mantığının projede birden fazla kez kolayca kullanılabilmesine olanak tanır.

Çalışma Mantığı
Projenin çalışma mantığı, ana program bloğu (Main [OB1]) ile özel fonksiyon bloğu (valve_2Pos [FB1]) arasındaki ilişkiye dayanır.

1. Ana Program (Main [OB1])
Main [OB1] bloğu, ana kontrol programıdır ve diğer blokları çağırır.

Fonksiyon Bloğu Çağrımı: Main [OB1]'deki Network 1, valve_2Pos [FB1] fonksiyon bloğunu çağırır.

Giriş ve Çıkış Bağlantıları:

Girişler:

EN: Bloğun etkinleştirilmesi.

Valve_Cmd: %I0.0 ile etiketlenmiş bir giriş. Bu, vananın açılması veya kapanması için gelen komut sinyalidir.

Opened_Sw: %I0.1 ile etiketlenmiş bir giriş. Bu, vananın tam açık olduğunu gösteren limit anahtarından gelen sinyaldir.

Closed_Sw: %I0.2 ile etiketlenmiş bir giriş. Bu, vananın tam kapalı olduğunu gösteren limit anahtarından gelen sinyaldir.

Çıkışlar:

ENO: Bloğun sorunsuz çalıştığını gösterir.

Cmd_State: %M10.0 ile etiketlenmiş bir çıkış. Bu, vana komutunun durumunu izler.

Valve_Alm_CmdDisc_Alarm: %M10.1 ile etiketlenmiş bir çıkış. Bu, komut ve vana pozisyonu arasında bir uyumsuzluk olduğunda bir alarmı tetikler.

2. Fonksiyon Bloğu (valve_2Pos [FB1])
Bu blok, 2 konumlu bir vana için özel kontrol lojiğini içerir. Bu yaklaşım, kodun tekrar kullanılabilir olmasını sağlar.

Network 1 (Durum Kontrolü):

%I0.0 (Valve_Cmd) girişi, vananın komut durumunu kontrol eder ve %M10.0 (Cmd_State) çıkışına bağlar. Bu, vanaya hangi komutun verildiğini izlemeyi sağlar.

Network 2 (Alarm Lojiği):

Bu ağ, bir TON (On-Delay Timer) kullanarak bir alarm durumunu yönetir.

Alarmın Tetiklenmesi:

Senaryo 1: Valve_Cmd aktif (vana açılma komutu verildi) ancak Opened_Sw (açık limit anahtarı) aktif değilse, zamanlayıcı saymaya başlar.

Senaryo 2: Valve_Cmd pasif (vana kapanma komutu verildi) ancak Closed_Sw (kapalı limit anahtarı) aktif değilse, zamanlayıcı saymaya başlar.

Zamanlayıcı İşlevi:

TON bloğunun IN girişi, yukarıdaki iki senaryodan biri doğru olduğu sürece aktiftir.

PT (Preset Time) girişi, Alarm_Tmr değişkeni tarafından belirlenir, varsayılan değeri 5 saniyedir (T#5S). Bu süre, vananın beklenen konumuna ulaşması için tanınan maksimum süredir.

Alarm Çıkışı: Zamanlayıcı, 5 saniyelik süreyi tamamladığında, Q çıkışı aktif olur ve %M10.1 (Valve_Alm) alarm çıkışını tetikler. Bu, vananın komut verildiği halde beklenen pozisyona zamanında ulaşamadığı anlamına gelir. Bu durum, vananın mekanik olarak sıkışması, elektrik arızası veya bir limit anahtarının arızalanması gibi sorunları işaret eder.

3. Veri Bloğu (valve_2Pos_DB [DB2])
Fonksiyon bloğu, valve_2Pos_DB adlı bir instance data block (örnek veri bloğu) ile ilişkilidir.

Veri Depolama: Bu veri bloğu, fonksiyon bloğunun her çağrımı için giriş/çıkış değerlerini, statik değişkenleri ve zamanlayıcı değerlerini saklar. Bu, her bir vana örneğinin kendi verisine sahip olmasını ve birbirlerinden bağımsız çalışmasını sağlar. Örneğin, Alarm_Tmr değişkeni bu veri bloğunda tutulur, böylece her vana için farklı bir alarm süresi ayarlanabilir.
