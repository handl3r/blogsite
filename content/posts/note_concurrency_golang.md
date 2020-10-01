---
title: "Note_concurrency_golang"
date: 2020-10-01T15:58:55Z
draft: false
categories: [Golang-Basic, Draft]
---
----------------------Noteeeeeeeee--------------

1. Os schedualer mechanics  
các process tương tự như container, chứa các context cần thiết cho chương trình chạy.các process chạy thực chất chính là các pe của nó chạy.
path execution là 1 đơn vị thực thi. Ví dụ như 1 instruction mã máy chẳng hạn. Điều này còn phụ thuộc vào mức độ trừu tượng mà ta muốn.
Os schedualer quan tâm đến các path execution này. Ví dụ 1 core cần chạy 10000 path execution thì nó sẽ phải tính toán cho mỗi path execution 1 slice of time trên core đó.Sau đó lại phải chuyển context cho pe khác chạy. Với mức os thì điều này rất tốn thời gian bởi nó ko biết pe đó làm gì mà chỉ biết nó đang chạy. Chính vì vậy mà càng nhiều pe thì chi phí cho context switch càng lớn.
- Khi có 1 core thì schedualer hoạt động oke Nhưng khi có nhiều core thì có vấn đề khi mà có những pe ở trạng thái runable nhuưng lại ko đcchạy trên 1 core đang idle.
1 path execution có thể ở 3 trạng thái là running, runnable, waiting
less is more: nếu có ít thread hơn thì mỗi thread sẽ có nhiều time hơn của cpu
tưởng tượng 1 mt có 4 core, mỗi core là 2 cache L1 L2, 4 core dùng chung L3 và main memory. Khi đó, giả dử từ luông 1 đang chạy trên cache 1 sinh ra luồng 2. Nếu như 3 core còn lại đang bận thì ta nên cân nhắc việc pull of luồng chính ra khỏi C1 và đâỷ luồng con mới vào C1 để chạy. Ưu điểm là luồn con mới có thể tận dụng các data trên cache L1 và l2 ở đó. Tuy nhiên nếu như có 1 core nào đó rảnh thì ta có thể chạy luồng con mới trên core  mới mà không caanf chờ và luồng chính vaanc chạy được. Nhanh nhưng nếu muốn dùng data trên cache L1, L2 của C1 thì ta sẽ phải duplicate nó sang cache L1 L2 của core khác mà luồng con sắp chạy.
Trên mỗi core người ta cũng tạo ra các queue chứa những runable path execution
Xem thêm trong ảnh IOCP
2. Go schedualer mechanics  
Go schedualer build on Go runtime, Goruntime build on Go application -> goschedualer chạy trên user mode.
Phân biệt kernel mode và user mode: kernel mode là chế độ mà code chạy trên kernel mode có thể làm bất cứ thứ gì nó muốn. Ví dụ như os, driver ()đó là lí do mà driver có thể làm hỏng os.
Còn usermode thì sẽ có những hạn chế.
trước kia khi lập trình viên muốn tự lập lịch trong chương trình của mình (ko có go schedualer) thì có vấn đề là nếu  pe có time execution trên processor(ảo) thì nó sẽ ko bỏ quyền thực thi đó dù nó ko làm gì(tính ích kỉ). Go scheduler thay đổi điều đó bằng cách nó chính là người điều phối lập lịch chạy và ko thể dự đoán. Tất nhiên là quyền của nó là dươi quyền scheduler của os có nghĩa là nó phải hợp tác với scheduler os. Tuy nhiên với việc ta trừu tượng trong chương trình Go bằng các goroutine và go schedualer thì có thể coi go schedualer là tối cao rồi(tính trong suốt).
- Thông số tham khảo lại : runtime.GOMAXPROCS


3. Data races  
// Thực tế goruntine chạy trên 1 lõi thì là concurrency nhưng nếu máy có nhiều hơn 1 lõi thì nó lại vừa chạy conccurency trên từng core, vừa parallel trên nhiều core. Giống như ta đã bàn luận ở trên, nếu máy chỉ có 1 nhân hoặc chỉ có 1 nhân rảnh thì goruntime chỉ có thể thực hiện trên nhân đó, lúc đó la concurrency. Tuy nhiên nếu máy có nhiều nhân mà có nhiều nhân rảnh khác thì sẽ thực thi parallel. Thông số này được golang ảo hóa, trừu tượng hóa bằng logical processor có thể set bằng runtime.GOMAXPROCS(). Default thì thông số này là số nhân của máy, Nếu ta set thông số này là 1 thì chắc chắc sẽ thực thi conccurrency hoàn toàn. Nếu ta để mặc định với nhiều nhân hoặc chỉ đinhn nhiều nhân thì máy sẽ thực thi với cả parallel.
Code: https://play.golang.org/p/tB2nMflZVGd
Với code này thì ta thấy nếu set GOMAXPROCS là 1 thì kết quả luôn là 30000. Nhưng nếu set lên trên 1 thì xảy ra data race, sẽ có những lần thực thi kết quả khác 30000. 

Cần phân biệt khi nào ta phải cân nhắc về vấn đêf synchronization hay orcheration issues. Synchronization là khi mà nhiều core truy cập vào cùng 1 vùng nhớ. Còn orcheration chính là khi ta cần xử lí conccurrency.
3.1 Vấn đề về cache coherency và false sharing   
Thực tế synchronization có nhiều vấn đề kể cả khi mà các pe không thực hiện truy cập vào cùng 1 vùng nhớ . Quay lại ví dụ về 4 core tương ứng với 4 cacche L1 L2 và dùng chung L3. Giải sử ta có 1 global variable. Biến này được đưa vào cache tại L1, L2 của cả 4 core.Khi đó ta gặp vấn đề về cache cohenrency. Để xử lí vấn đề này thì ta có cơ chế trong phần cứng. Khi mà core 1 update biến đó thì nó các giá trị copy value của 3 core còn lại trong cache đc đánh dấu là dirty. Giả sử như khi core 2 thực thi, nó thấy dirty nên sẽ thực hiện load lại giá trị từ main memory. -> xử lí được . Tuy nhiên lại có trường hợp. Giả sử như có 1 global var là 1 mảng 4 phần tử mà mỗi core thay đổi 1 phần tử. Có vẻ vấn đề không xảy ra. và ta không cần đánh dấu dirty như trường hợp trước. Tuy vậy không phải. Cache tường được load theo cache line. Trong tường hợp trên thì cả mảng đó nằm  trong 1 cache line. Nếu ko đánh dấu dirty thì data trong line ko đc load lại sẽ là data cũ. Vấn đề này là false sharing. Để khắc phục vấn đề này thì ta vẫn tiếp tục đánh dấu dirty như với trường hợp bên trên
3.2 Synchronization with atomic function  
ví dụ: code: https://play.golang.org/p/TiRueoGdHtA
Trong ví dụ này nếu ta bỏ dòng in value ra stdout thì kết quả hầu như là 6(nếu tăng số lần lặp lên mới gặp vấn đề về data race thường xuyên ). Nhưng khi ta thêm dòng print vào thì câu chuyện khác.khi chạy đến dòng print thì gorountine sẽ bị context switch(bởi gọi đến systemcall ). Khi đó nó tạm thời chưa ghi đè giá trị tiếp theo vào được, goroutine khác cũng vậy, cho đến khi nó thực thi lại sau context switch thì mới thay đổi giá trị biến count. -> kết quả là 2.
để giải quyết trường hợp này ta có thể dùng thư viện sync/atomic.Như sau:
code: https://play.golang.org/p/85hany5iEtw (nếu muốn có thể lặp 1 triệu lần cũng oke)
Nhưng có những trường hợp ta muốn giữ lại logic trong code như bên trên thay vì dùng atomic thì sao ?
--- Đọc thêm về goroutine: https://stackoverflow.com/questions/41045362/what-happens-with-cpu-context-when-goroutines-are-switching
https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part1.html
3.3 Synchronization with mutexes
Mutex, ReadMutex, WriteMutex
3.4 Race detecion
3.5 Map data race
3.6 interface-based race

4. Channel

4.1 Signaling semantics
Khi không muốn độ trễ cao thì dùng buffered channel, khi muốn đảm bảo(tính guarantees)thì dùng unbuffered channel.

- COst của sự đảm bảo(khi ta dùng unbuffered channel) là việc không thể dự đoán, ko biết đc trễ bao nhiêu. Đơn giản là khi ta gửi signal đi thì sẽ bị block cho đến khi có goroutine nhận nó. -> ko biết trễ bao lâu
- Cost của ko bị trễ(khi ta dùng buffered channel) là tính đảm bảo yếu đi.
1 channel có thể ở 3 trạng thái:
nil	: gửi và nhận đều bị block
open	: gửi và nhận đc cho phép
closed	: gửi ko đc, nhận thì đc

ta có thể signal without data hoặc với data. Nếu có data thì chỉ signal đến 1 goroutine khác đc nhưng nếu ko có data thì có thể signal đến nhiều goroutine khác

Basic Pattern: 

4.2 Basic pattern(wait for task)
Code: https://play.golang.org/p/RKR5S_yqXPB
trong ví dụ trên, 2 goroutine chạy song song với nhau. chạy nhiều lầ sẽ thấy lúc thi in ra receive trước lúc thì in ra send trước. Tuy vậy điều này chỉ nói về việc hành vi in nào thực hiện trước. Thực tế thì hành vi nhận sẽ thực hiện trước send vài nano giây.
- đầu tiên từ main goroutine thì tạo ra goroutine khác
- đến dòng 13 thì thực hiện hành vi receive trên channel nhưng lại không có data nên gorootine này bị block, chuyển qua trạng thái waiting.
- Ở 1 core khác thì sau khi sleep 1 vài milli second thif goroutine sẽ send signal với data vào channel. Đoạn xử  lí atomic duy nhất ở đây chính là lúc mà receive và send gặp nhau. Còn lại thì 2 goroutine chạy độc lập và song song nên không thể đoán biết dược câu lệnh nào in ra trước.
Sau đoạn send and receive gặp nhau thì chắc chắn 1 điều là receive xong rồi mới send vì phải receive rồi thì goroutine thực hiện hiện send mới hết block
5.1 Basic pattern(wait for result)
Code: https://play.golang.org/p/eUSfGDMbRZr
Tương tự như trên.
5.2 Basic pattern (wait for finished)
Chú ý: wait group chính là 1 kiểu signal without data
Ta có thể sử dụng channel với empty struct chính là signal without data
Code: https://play.golang.org/p/vFZr8KV-TWG
bằng cách close channel thì mọi goroutine receive từ channel đó sẽ nhận về tham số thứ 2 là bool, true nếu có data trả về và false nếu không

5.3 Pooling pattern
Code: https://play.golang.org/p/BjHD5q4IlQO
Dùng range lặp qua channel giúp goroutine lấy data từ channel cho đến khi channel close thì nó sẽ thoát khỏi vòng lặp.


5.4 Fanout
Không ứng dụng tốt trong web service vì số lượng goroutine có thể tăng lên chóng mặt.
Nhưng trong những ứng dụng CLI hay background thì oke.
Code: https://play.golang.org/p/V2NHWfybjL3
5.5 fan out semaphore pattern
tuong tự như fan out nhưng ta giới hạn số lượng goroutine xử lí công việc trong 1 thời điểm
Code: https://play.golang.org/p/PCylnbGZ_g6
cost: độ trễ khi mà sem full và những goroutine khác phải chờ.
5.6 Drop pattern
Code: https://play.golang.org/p/NYwboXkeLjW
đại loại là ta tạo ra 1 channel chứa công việc mà các machine bên ngoài đẩy việc vào, các workers lấy việc cũng từ channel đó để thực hiện.
Tương tự như bài toán khi các requesst đến từ mạng bên ngoài và ta phải xử lí công việc mà tài nguyên chỉ có hạn. Khi đó có những thời điểm mà các workers đã hoạt động full thì ta sẽ không chấp nhận xử lí công việc củâ mạng ngoài nữa mà từ chối chúng.
5.7 Cancellation pattern
Code: https://play.golang.org/p/wrn87lrBJmb
đại loại là ta muốn timeout hoawcjc chờ kết quả trên nhiều channel để lấy kết quả đến sớm nhất. Chẳng hạn khi t query 2 nguồn dữ liệu và chỉ muốn lấy nguồn dữ liệu trả về nhanh nhất.  
6. Conccurrency pattern

