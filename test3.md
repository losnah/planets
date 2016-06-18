Enter file contents here

![1](https://github.com/losnah/planets/blob/master/1.png)

![2](https://github.com/losnah/planets/blob/master/2.png)

####**3-4. 상세 설계**

#####**3-4-1. 모바일티켓 주요 모듈 구조**

![3](https://github.com/losnah/planets/blob/master/3.png)

######**◉ NFC 태그 모듈**

```
public class WriteActivity extends ActionBarActivity {

   private NfcAdapter nfcAdapter;
   private PendingIntent pendingIntent;

   public static final int TYPE_TEXT = 1;
   public static final int TYPE_URI = 2;

   EditText writeText;

   @Override
   protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_write);

if (savedInstanceState == null) {
         getSupportFragmentManager().beginTransaction()
               .add(R.id.container, new PlaceholderFragment()).commit();
      }

      writeText = (EditText) findViewById(R.id.writeText);

      // NFC 관련 객체 생성
      nfcAdapter = NfcAdapter.getDefaultAdapter(this);
      Intent intent = new Intent(this, getClass())
            .addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP);
      pendingIntent = PendingIntent.getActivity(this, 0, intent, 0);
   }
```
```
// NFC 태그 스캔시 호출되는 메소드
   @Override
   protected void onNewIntent(Intent intent) {
      super.onNewIntent(intent);

      if (intent != null) {
         processTag(intent); // processTag 메소드 호출
      }
   }
```
```
// onNewIntent 메소드 수행 후 호출되는 메소드
   private void processTag(Intent intent) {
      // EditText에 입력된 값을 가져옴
      String s = writeText.getText().toString();

      // 감지된 태그를 가리키는 객체
      Tag detectedTag = intent.getParcelableExtra(NfcAdapter.EXTRA_TAG);


      // 아무것도 입력받지 않으면 태그에 쓰지 않음
      if (s.equals("")) {
         Toast.makeText(getApplicationContext(), "내용을 입력해주세요.", 1000).show();
      }

      // 입력받은 값을 감지된 태그에 씀
      else {
         NdefMessage message = createTagMessage(s, TYPE_TEXT);
         writeTag(message, detectedTag);
      }
   }
```
```
// 감지된 태그에 NdefMessage를 쓰는 메소드
   public boolean writeTag(NdefMessage message, Tag tag) {
      int size = message.toByteArray().length;
      try {
         Ndef ndef = Ndef.get(tag);
         if (ndef != null) {
            ndef.connect();
            if (!ndef.isWritable()) {
               return false;
            }
            if (ndef.getMaxSize() < size) {
               return false;
            }
            ndef.writeNdefMessage(message);
            Toast.makeText(getApplicationContext(), "쓰기 성공!", 1000).show();
          }
          else { Toast.makeText(this, "포맷되지 않은 태그이므로 먼저 포맷하고 데이터를 씁니다.",
                  Toast.LENGTH_SHORT).show();
            NdefFormatable formatable = NdefFormatable.get(tag);
            if (formatable != null) {
               try {
                  formatable.connect();
                  formatable.format(message);
               } catch (IOException ex) {
                  ex.printStackTrace();
               }
            }
            return false;
         }
      } catch (Exception ex) {
         ex.printStackTrace();
         return false;
      }
      return true;
   }
```
```
// NFC 태그 ID를 리턴하는 메소드
   public static final String CHARS = "0123456789ABCDEF";
   public static String toHexString(byte[] data) {
      StringBuilder sb = new StringBuilder();
      for (int i = 0; i < data.length; ++i) {
         sb.append(CHARS.charAt((data[i] >> 4) & 0x0F)).append(
               CHARS.charAt(data[i] & 0x0F));
      }
      return sb.toString();
   }
```
```
// onNewIntent 메소드 수행 후 호출되는 메소드
   private void processTag(Intent passedIntent) {
      Parcelable[] rawMsgs = passedIntent
            .getParcelableArrayExtra(NfcAdapter.EXTRA_NDEF_MESSAGES);
      if (rawMsgs == null) { return; }
      // 참고! rawMsgs.length : 스캔한 태그 개수
      Toast.makeText(getApplicationContext(), "스캔 성공!", 1000).show();
      NdefMessage[] msgs;
      if (rawMsgs != null) {
         msgs = new NdefMessage[rawMsgs.length];
         for (int i = 0; i < rawMsgs.length; i++) {
            msgs[i] = (NdefMessage) rawMsgs[i];
            showTag(msgs[i]); // showTag 메소드 호출
         }
      }
   }
```
```
// NFC 태그 정보를 읽어들이는 메소드
private int showTag(NdefMessage mMessage) {
   List records = NdefMessageParser.parse(mMessage);
   final int size = records.size();
      for (int i = 0; i < size; i++) {
      ParsedRecord record = records.get(i);
         int recordType = record.getType();
      String recordStr = ""; // NFC 태그로부터 읽어들인 텍스트 값
      if (recordType == ParsedRecord.TYPE_TEXT) {
         recordStr = "TEXT : " + ((TextRecord) record).getText();
      } else if (recordType == ParsedRecord.TYPE_URI) {
         recordStr = "URI : " + ((UriRecord) record).getUri().toString();
      }
         readResult.append(recordStr + "\n"); // 읽어들인 텍스트 값을 TextView에 덧붙임
   }
   return size;
}
```
#####**3-4-2. 서버 주요 모듈 구조**
 
 ![4](https://github.com/losnah/planets/blob/master/4.png)

######**◉ 단말기 검색 모듈**
```
class DataTerminal {
   int room; //호실 번호
   int seat;  //좌석 번호
   int code; //좌석 고유 번호

   void Set_RoomNumber(); // 호실 번호 설정자
   int Get_RoomNumber();  // 호실 번호 접근자
   int Get_SeatNumber();
   void Set_SeatNumber();
   int Room_number_search(); // 호실 번호 찾기
   int Seat_number_search();  // 좌석 번호 찾기
};
```
```
// 호실 번호 찾기
int Room_number_search() {
   for(int i = 0; i < 8; i++) {
      if(Get_RoomNumber = i)
         code = i * 100;
   }
}
```
```
// 좌석 번호 찾기
int Seat_number_search() {
   for(int i = 0; i < 72; i++) {
      if(Get_SeatNumber = i)
         code = code + i;
   }
}
```

