[English version](https://github.com/unityvn/how-to-detect-region-unity/wiki/English-version)
# 🌍 How to detect regions in Unity
## ❓Đặt vấn đề
- Gần đây mình có nhận được một task đó là: `Đổi logo/background cho màn hình loading nếu user đó ở thị trường Nhật Bản 🇯🇵`.
- Sau khi nhận task mình đã rất tự tin vì nghĩ rằng việc này rất đơn giản, chỉ cần sử dụng `RegionInfo.CurrentRegion.TwoLetterISORegionName` để lấy ra vùng hiện tại của thiết bị người dùng là xong. Nhưng kết quả khi chạy game và Debug.Log ra thì lại không như mình mong muốn. Rõ ràng mình đang ở vùng Việt Nam mà kết quả lại rất lung tung. Mình đã thử trên các môi tường khác nhau như Editor (Windows/macOS), iOS, Android nhưng kết quả thường trả về `US` hoặc `IV` mặc dù trong cài đặt thiết bị của tôi để vùng là Việt Nam.
- Research một vòng mình cũng tìm được một vài người có cùng vấn đề với mình

   - [RegionInfo.CurrentRegion returns wrong region](https://discussions.unity.com/t/regioninfo-currentregion-returns-wrong-region/938267)
   - [C# CurrentCulture always return “en-US”](https://discussions.unity.com/t/c-currentculture-always-return-en-us/630359)
   - [CultureInfo.CurrentCulture always returning Invariant Language (Invariant Country)](https://discussions.unity.com/t/cultureinfo-currentculture-always-returning-invariant-language-invariant-country/714652)

- Vì sao lại trả về kết quả sai như thế thì mình vẫn chưa tìm được lời giải thích.
## 💡Giải pháp

- Trong lúc bế tắc thì mình có thấy khá nhiều cách để xác định người dùng đang ở thị trường nào:
### 🧩 Cách 1: Xác định thông qua ngôn ngữ hệ thống
Ví dụ:
```csharp
    private void Start()
    {
        if (Application.systemLanguage == SystemLanguage.Japanese)
        {
            // Handle logic
        }
    }
```
✅ Ưu điểm:

- Dễ dàng sử dụng do đã tích hợp trong Unity.
- Không vướng mắc về quyền truy cập

❌ Nhược điểm

- Độ chính xác không cao do người dùng có thể thay đổi ngôn ngữ khác

### 🧩 Cách 2: Xác định thông qua vùng của thiết bị
Ví dụ: 
```csharp
    private void Start()
    {
        if (RegionInfo.CurrentRegion.TwoLetterISORegionName.Equals("JP"))
        {
            // Handle logic
        }
    }
```
Ví dụ: Sử dụng [system region unity](https://github.com/unity-package/system-region-unity) 
```csharp
    private void Start()
    {
        if (RegionHelper.GetCurrentRegion().TwoLetterISORegionName.Equals("JP"))
        {
            // Handle logic
        }
    }
```

✅ Ưu điểm:

- Dễ dàng sử dụng do đã tích hợp trong Unity.
- Không vướng mắc về quyền truy cập

❌ Nhược điểm

- Độ chính xác trung bình do người dùng có thể thay đổi vùng thiết bị (ít khi người dùng thực hiện việc này).
- Kết quả trả về không chính xác trong một số trường hợp (như đã đề cập ở trên). Tuy nhiên mình đã viết một thư viện xử lý vấn đề này và trả về kết quả rất chính xác trên Android/iOS
- Mọi người có thể sử dùng thư viện [system region unity](https://github.com/unity-package/system-region-unity) để lấy ra vùng hiện tại của thiết bị

### 🧩 Cách 3: Xác định thông qua địa chỉ IP nhà mạng

- Thông qua một số API, bạn có thể xác định vị trí của người dùng dựa trên địa chỉ IP của họ. Một số API phổ biến bao gồm:
  - [ip-api.com](http://ip-api.com/json/)
  - [ipinfo.io](https://ipinfo.io/json)
  - [ipwhois.app](https://ipwhois.app/json/)

- Ví dụ sử dụng [ipinfo.io](https://ipinfo.io/json):

```csharp
    void Start()
    {
        StartCoroutine(GetIPRegion());
    }

    IEnumerator GetIPRegion()
    {
        UnityWebRequest www = UnityWebRequest.Get("https://ipinfo.io/json");
        yield return www.SendWebRequest();

        if (www.result == UnityWebRequest.Result.Success)
        {
            Debug.Log(www.downloadHandler.text);
        }
        else
        {
            Debug.LogError("Error: " + www.error);
        }
    }
```

Kết quả thu được khi chạy game

<img width="348" alt="Screenshot 2025-06-11 at 22 48 17" src="https://github.com/user-attachments/assets/01ab7750-fc3f-4411-80dc-5f356d1008f1" />

✅ Ưu điểm
- Không cần quyền (Dễ dàng truy cập qua API của bên thứ ba).
- Có thể xác định quốc gia, vùng, thành phố (Thường chính xác đến cấp thành phố).

❌ Nhược điểm
- Cần kết nối Internet (phải có kết nối mạng để truy cập API).
- Phụ thuộc vào bên thứ ba (Nếu API không hoạt động hoặc thay đổi, thì kết quả trả về sẽ gặp vấn đề)
- Cần xử lý file json để lấy thông tin cần thiết
- Thông tin có thể bị sai lệch nếu người dùng fix VPN.

### 🧩 Cách 4: Xác định thông qua vị trí GPS (Global Positioning System)

✅ Ưu điểm
- Chính xác cao (Có thể xác định vị trí theo vĩ độ, kinh độ, độ cao)
- Hữu ích cho game/location-based (Phù hợp với các ứng dụng cần vị trí thực tế (AR games, tracking...)).

❌ Nhược điểm
- Yêu cầu quyền truy cập vị trí (Trên Android/iOS, cần người dùng cho phép).
- Tốn pin và tài nguyên (Đặc biệt khi theo dõi liên tục).
- Không khả dụng trong môi trường thiếu tín hiệu (Trong nhà, tầng hầm)

(Cách này có ai dùng rồi có thể contribute giúp mình phần ví dụ với nhé. Mình xin cảm ơn hehe!...)

## 📊 Tổng hợp Ưu và Nhược điểm các cách xác định vùng trong Unity

| 💡Phương pháp                          | ✅ Ưu điểm                                                                                        | ❌ Nhược điểm                                                                                      |
|-------------------------------------|--------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------|
| **Ngôn ngữ hệ thống**               | - Dễ sử dụng với `Application.systemLanguage`<br>- Không yêu cầu quyền truy cập                  | - Độ chính xác thấp vì người dùng có thể đổi ngôn ngữ không theo vị trí thực tế                   |
| **Vùng của thiết bị**               | - Dễ dùng với `RegionInfo`<br>- Không yêu cầu quyền<br>- Có thể cải thiện với thư viện bên ngoài | - Độ chính xác trung bình<br>- Có thể trả về sai giá trị trên một số thiết bị hoặc hệ điều hành   |
| **IP nhà mạng (qua API)**           | - Có thể xác định được chính xác quốc gia, vùng, thành phố<br>- Không cần quyền                  | - Cần kết nối Internet<br>- Phụ thuộc bên thứ ba<br>- Dễ bị sai nếu dùng VPN<br>- Phải xử lý JSON |
| **GPS**                             | - Chính xác cao<br>- Hữu ích cho các ứng dụng cần vị trí thực tế                                 | - Cần quyền truy cập vị trí<br>- Tốn pin<br>- Không hiệu quả trong nhà hoặc nơi tín hiệu yếu      |

> 🔗 Tham khảo thư viện hỗ trợ vùng thiết bị chính xác hơn: [system-region-unity](https://github.com/unity-package/system-region-unity)

## 📌 Kết luận
- Mỗi phương pháp đều có ưu và nhược điểm riêng, tùy thuộc vào yêu cầu cụ thể của ứng dụng mà bạn có thể chọn phương pháp phù hợp nhất.
- Trường hợp của mình thì mình đã chọn cách 2 vì:

    - Dễ dàng sử dụng, không lệ thuộc vào bên thứ ba, không yêu cầu quyền truy cập.
    - Độ chính xác trung bình nhưng vẫn đủ để đáp ứng yêu cầu vì game của mình cho phép người dùng chơi offline.
    - Người dùng cũng ít khi thay đổi vùng thiết bị (hoặc để vùng tự động) vì thường khi thay đổi vùng sẽ bị sai giờ trong máy và có thể không sử dụng được internet.
