# Ghi chú phân tích · Day 04

Họ và tên: Vũ Quang Huy

MSSV: 2A202603782

## 1. Case được chọn

- ID case: A1
- Câu trả lời của agent: Agent báo incident GitHub Actions đã resolved và Cloudflare Spectrum Phoenix ở trạng thái monitoring. Tuy nhiên, nó liệt kê năm incident GitHub dù câu hỏi cần đối chiếu dịch vụ, đồng thời gắn URL `https://stspg.io/7pwv28hvkmz8` cho cả incident `t89g2q1800fr` và `h02n9572mx9y`. URL đúng của `t89g2q1800fr` trong tool result là `https://stspg.io/lv4q3xm9r0rk`. Câu trả lời cuối không hoàn tất vòng kiểm tra nguồn.
- Điểm ở lần chạy 1: 0/100; trạng thái `max_steps`.

## 2. Bằng chứng trong trace

Dán đoạn trace chứng minh lỗi và nêu nguyên nhân có thể xảy ra.

```json
[
  {
    "event": "tool",
    "step": 2,
    "name": "get_incident_details",
    "arguments": {
      "provider": "github",
      "incident_id": "t89g2q1800fr"
    },
    "result": {
      "ok": true,
      "provider": "github",
      "captured_at": "2026-11-09T03:50:47.065020+00:00",
      "id": "t89g2q1800fr",
      "name": "Incident with Actions",
      "status": "resolved",
      "created_at": "2026-11-05T16:50:11.086Z",
      "updated_at": "2026-11-05T17:42:35.455Z",
      "components": ["Actions"],
      "updates": [
        {
          "status": "resolved",
          "created_at": "2026-11-05T17:42:35.425Z",
          "body": "This incident has been resolved. Thank you for your patience and understanding as we addressed this issue. A detailed root cause analysis will be shared as soon as it is available.",
          "truncated": false
        },
        {
          "status": "monitoring",
          "created_at": "2026-11-05T17:04:46.807Z",
          "body": "The degradation affecting Actions has been mitigated. We are monitoring to ensure stability.",
          "truncated": false
        },
        {
          "status": "investigating",
          "created_at": "2026-11-05T16:50:11.189Z",
          "body": "We are investigating reports of degraded performance for Actions",
          "truncated": false
        }
      ],
      "updates_total": 4,
      "url": "https://stspg.io/lv4q3xm9r0rk"
    }
  },
  {
    "event": "tool",
    "step": 2,
    "name": "get_incident_details",
    "arguments": {
      "provider": "github",
      "incident_id": "h02n9572mx9y"
    },
    "result": {
      "ok": true,
      "provider": "github",
      "captured_at": "2026-11-09T03:50:47.065020+00:00",
      "id": "h02n9572mx9y",
      "name": "Incident with Actions",
      "status": "resolved",
      "created_at": "2026-11-04T17:40:33.675Z",
      "updated_at": "2026-11-07T01:10:18.506Z",
      "components": ["Actions"],
      "updates": [
        {
          "status": "resolved",
          "created_at": "2026-11-04T18:16:08.361Z",
          "body": "Please refer to the combined summary in this related incident: https://www.githubstatus.com/incidents/xr3myzj7ms51",
          "truncated": false
        },
        {
          "status": "monitoring",
          "created_at": "2026-11-04T18:07:51.114Z",
          "body": "The degradation affecting Actions has been mitigated. We are monitoring to ensure stability.",
          "truncated": false
        },
        {
          "status": "investigating",
          "created_at": "2026-11-04T17:40:33.887Z",
          "body": "We are investigating reports of degraded availability for Actions",
          "truncated": false
        }
      ],
      "updates_total": 4,
      "url": "https://stspg.io/7pwv28hvkmz8"
    }
  },
  {
    "event": "validation_missing_source",
    "step": 3,
    "urls": ["https://stspg.io/lv4q3xm9r0rk"]
  },
  {
    "event": "validation_missing_source",
    "step": 4,
    "urls": ["https://stspg.io/lv4q3xm9r0rk"]
  },
  {
    "event": "validation_missing_source",
    "step": 5,
    "urls": ["https://stspg.io/lv4q3xm9r0rk"]
  },
  {
    "event": "validation_missing_source",
    "step": 6,
    "urls": ["https://stspg.io/lv4q3xm9r0rk"]
  },
  {
    "event": "validation_missing_source",
    "step": 7,
    "urls": ["https://stspg.io/lv4q3xm9r0rk"]
  },
  {
    "event": "validation_missing_source",
    "step": 8,
    "urls": ["https://stspg.io/lv4q3xm9r0rk"]
  }
]
```

Diễn giải trace: Ở step 1, agent kiểm tra status GitHub Actions và Cloudflare Spectrum. Ở step 2, agent đọc chi tiết năm incident GitHub và một incident Spectrum. Bản trả lời cuối gắn URL `https://stspg.io/7pwv28hvkmz8` cho hai incident GitHub, trong khi trace ghi URL của `t89g2q1800fr` là `https://stspg.io/lv4q3xm9r0rk`. Từ step 3 đến step 8, validation đều báo thiếu URL này nhưng agent tiếp tục trả lời mà không sửa. Run kết thúc với `max_steps`; grader chấm 0 vì agent hết số bước trước khi hoàn tất.

Nguyên nhân có thể xảy ra: model lấy URL của incident `h02n9572mx9y` gắn nhầm cho `t89g2q1800fr`, rồi không sửa theo yêu cầu validation. Validation hiện chỉ kiểm tra URL có xuất hiện trong toàn câu trả lời, chưa kiểm tra URL có được gắn với đúng incident ID hay không; việc yêu cầu sửa lặp lại cũng tiêu tốn hết `max_steps`.

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: Trong `agent.py`, lưu cặp incident ID/URL từ các lần gọi `get_incident_details`. Trước khi trả lời, kiểm tra URL nguồn và yêu cầu URL chính xác nằm cùng dòng với incident ID nếu câu trả lời nhắc ID đó. Khi thiếu hoặc ghép sai nguồn, trace ghi `validation_missing_source` và agent yêu cầu model sửa với ID và URL cần ghép.
- Vì sao chọn thay đổi này: Trace A1 lần đầu cho thấy model dùng URL của incident `h02n9572mx9y` cho `t89g2q1800fr`, bỏ URL đúng của incident này, rồi lặp lại câu trả lời đến hết 8 bước. Cải tiến nhằm phát hiện và sửa lỗi trích dẫn trước khi agent kết thúc.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 75.91/100 (22/22 scored) | 77.05/100 (22/22 scored), tăng 1.14 điểm |
| Điểm case đã chọn | A1: 0/100 (`max_steps`) | A1: 55/100 (`completed`), tăng 55 điểm |
| Số model calls | 55 agent calls + 22 judge calls | 54 agent calls + 22 judge calls |
| Token, chi phí nếu có | 109,221 tổng token: 75,844 agent + 33,377 judge; chi phí không có trong report | 99,150 tổng token: 57,674 agent + 41,476 judge; chi phí không có trong report |

- Lỗi đã giải quyết: A1 hoàn tất thay vì lặp lại kiểm tra nguồn đến `max_steps`; tất cả 22 case đều được chấm. Các lần chạy dùng cùng provider/model (`openai`/`gpt-4o-mini`), judge, `max_steps=8` và dataset `core`.
- Lỗi còn lại hoặc mới xuất hiện: A1 vẫn được 55 vì một số link trong tiêu đề incident không khớp ID dù URL đúng xuất hiện ở dòng nguồn riêng. Điểm A5 còn 55 do không thể kết luận không có incident trong cả năm 2025 từ dữ liệu giới hạn. H2 (50) thiếu `get_status`; H7 (50) suy luận Cloudflare operational khi tool timeout; H8 (60) chưa nêu rõ giới hạn 50 incident. H1 và H6 (60) bị giới hạn điểm do cần gọi tool ở ít nhất hai model steps khác nhau. Một số case giảm điểm giữa hai lần chạy: E3 100→85, A4 95→80, H7 70→50 và H8 75→60.

## 5. Giới hạn

Điểm được ước lượng bởi LLM judge và có thể chịu ảnh hưởng của model hoặc cách diễn giải rubric; ở đây agent và judge dùng cùng model. Snapshot cố định tại 2026-11-09, nên không phản ánh status trực tiếp hiện nay; tools cũng chỉ tìm trong 50 incident gần nhất và không đủ để chứng minh toàn bộ lịch sử. Mỗi phiên bản chỉ được chạy một lần, vì vậy thay đổi điểm không chứng minh ổn định hay quan hệ nhân quả; một số case tăng trong khi các case khác giảm.
