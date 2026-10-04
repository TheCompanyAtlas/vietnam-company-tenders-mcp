# Vietnam Company Registry & Government Tenders MCP

Connect your AI assistant to Vietnamese company registry and National E-Procurement System data through [The Company Atlas](https://thecompanyatlas.com) and the Model Context Protocol.

Tender data is sourced from [muasamcong.mpi.gov.vn](https://muasamcong.mpi.gov.vn).

Search:

- Registered enterprises by English or Vietnamese name, MST (tax identification number), enterprise type, VSIC industry and address
- Company profiles with legal representative and tender history — the MST links each company to the tenders it bid on and won
- Procurement notices, plans, projects, contracts, quote requests, and results
- Procuring entities and investors
- Tender packages and budgets
- Participants, organization codes, and award names

The MCP connection is provided by The Company Atlas. This repository does not run a separate server.

## Connect

```text
https://thecompanyatlas.com/mcp/vietnam
```

Add this URL as a custom MCP connector in Claude, Cursor, or another MCP-compatible client. Sign in with Google when prompted.

- Transport: Streamable HTTP
- Auth: Google OAuth
- Docs: [thecompanyatlas.com/tools/vietnam-company-tenders](https://thecompanyatlas.com/tools/vietnam-company-tenders)

## Tiếng Việt

Vietnam Company Registry & Government Tenders MCP cho phép trợ lý AI tra cứu thông tin doanh nghiệp Việt Nam và dữ liệu từ Hệ thống mạng đấu thầu quốc gia.

Bạn có thể:

- Tra cứu doanh nghiệp theo tên tiếng Anh hoặc tên tiếng Việt, mã số thuế (MST), loại hình doanh nghiệp, ngành nghề (VSIC) và địa chỉ
- Xem thông tin doanh nghiệp: người đại diện theo pháp luật, ngày thành lập, trạng thái và lịch sử tham gia, trúng thầu
- Tra cứu thông báo mời thầu, kế hoạch lựa chọn nhà thầu, dự án, hợp đồng, yêu cầu báo giá và kết quả lựa chọn nhà thầu
- Tìm kiếm theo tên hoặc mã thông báo, mã gói thầu, mã dự án và mã hợp đồng
- Lọc theo bên mời thầu, nhà đầu tư, lĩnh vực, tỉnh thành, trạng thái và ngày công khai
- Xem danh sách gói thầu, giá trị dự toán, nhà thầu tham dự và đơn vị trúng thầu
- Tìm lịch sử tham gia hoặc trúng thầu theo tên và mã doanh nghiệp

Để kết nối, hãy thêm URL sau vào Claude, Cursor hoặc ứng dụng hỗ trợ MCP:

```text
https://thecompanyatlas.com/mcp/vietnam
```

Đăng nhập bằng Google khi được yêu cầu. Sau khi kết nối, bạn có thể thử:

- *"Tra cứu doanh nghiệp có mã số thuế 0110399637 và các gói thầu đã trúng."*
- *"Tìm các thông báo mời thầu thiết bị y tế được công khai trong năm 2025."*
- *"Liệt kê các gói thầu của bên mời thầu này và tổng hợp giá trị."*
- *"Tìm các gói thầu có sự tham gia của Viettel và xác định đơn vị trúng thầu."*
- *"Hiển thị các nhà thầu và giá trị dự toán của hồ sơ đấu thầu này."*

## Tools

### `Vietnam_search_companies`

Search Vietnamese enterprises. Filters are combined with AND logic; include at least one.

| Parameter | Type | Description |
|---|---|---|
| `q` | string, optional | Name substring, English or Vietnamese (with diacritics), 3–120 characters |
| `entityType` | string, optional | Enterprise type substring, e.g. `Joint Stock` |
| `vsic` | string, optional | VSIC industry code prefix, 2–5 digits, e.g. `4100` |
| `address` | string, optional | Head-office address substring, e.g. `Đà Nẵng` |
| `active` | boolean, optional | Filter by active status |
| `establishedFrom` / `establishedTo` | date, optional | Establishment date range, `YYYY-MM-DD` |
| `page` / `limit` | integer, optional | Pagination, `limit` up to 50 (default 20) |

Returns `{ data, total, page, limit }`. The company `id` is `VN` + the MST.

### `Vietnam_get_company`

Full profile by MST (e.g. `0110399637`, with or without the `VN` prefix): former names, short name, legal representative, contact email, and a tender summary — participation count, award count and the 20 most recent participations.

| Parameter | Type | Description |
|---|---|---|
| `mst` | string | MST tax identification number |

### `Vietnam_list_tenders`

Search Vietnam procurement records. Filters are combined with AND logic; `query` searches the record title and common notice, plan, project, bid, and contract numbers.

| Parameter | Type | Description |
|---|---|---|
| `recordType` | enum, optional | Procurement record type; see values below |
| `query` | string, optional | Title or procurement identifier; minimum 2 characters |
| `procuringEntity` | string, optional | Partial procuring-entity name |
| `participant` | string, optional | Partial participant organization or award name |
| `provinceCode` | string, optional | Exact province code in the record |
| `bidField` | string, optional | Partial bid-field value |
| `status` | string, optional | Partial status value |
| `publicDateFrom` | date, optional | Earliest publication date, `YYYY-MM-DD` |
| `publicDateTo` | date, optional | Latest publication date, `YYYY-MM-DD` |
| `page` | integer, optional | Page number, minimum `1`, default `1` |
| `limit` | integer, optional | Results per page, `1`–`100`, default `20` |

Supported record types:

- `NOTICE_CONTRACTOR`
- `PLAN_PROJECT`
- `BID_PROJECT`
- `CONTRACT`
- `QUOTE_REQUEST`
- `CONTRACTOR_INPUT_RESULT_OTHER`
- `CONTRACTOR_SELECTION_PLAN`
- `CONTRACT_PUBLISH_FRAME`
- `PRE_NOTICE_CONTRACTOR`
- `SHOPPING_RESULT`
- `INTEREST_NOTICE`

Returns `{ data, total, page, limit }`, ordered by publication date descending. Value fields are returned as decimal strings to preserve precision.

Example:

```json
{
  "recordType": "NOTICE_CONTRACTOR",
  "query": "medical equipment",
  "provinceCode": "79",
  "publicDateFrom": "2025-01-01",
  "limit": 20
}
```

### `Vietnam_get_tender`

Fetch one procurement record after selecting its `id` from a search result.

| Parameter | Type | Description |
|---|---|---|
| `id` | string | Exact Vietnam procurement record id |

Returns the record's identifiers, publication and decision dates, investor and procuring entity, procurement status and method, value fields, locations, packages, and participants. Returns `null` for an unknown id.

Example:

```json
{
  "id": "record-id-from-search"
}
```

### `Vietnam_search_participants`

Search organizations named in Vietnam procurement records.

| Parameter | Type | Description |
|---|---|---|
| `query` | string | Partial organization name, organization code, or award name |
| `awardedOnly` | boolean, optional | Only rows with an award name; default `false` |
| `page` | integer, optional | Page number, minimum `1`, default `1` |
| `limit` | integer, optional | Results per page, `1`–`100`, default `20` |

Returns `{ data, total, page, limit }`. Each participant includes its related procurement-record summary.

Example:

```json
{
  "query": "Viettel",
  "awardedOnly": true,
  "limit": 20
}
```

## Example prompts

- *"Find active joint stock companies in Đà Nẵng in building construction (VSIC 4100)."*
- *"Show the profile and tender wins for MST 0110399637."*
- *"Find Vietnamese contract notices for medical equipment published in 2025."*
- *"Show procurement records from this procuring entity and summarize their values."*
- *"Find tenders involving this supplier and identify awarded records."*
- *"Get the packages and participants for this procurement record id."*

## Setup — Claude

1. Go to [claude.ai/customize/connectors](https://claude.ai/customize/connectors)
2. Click **Add custom connector**
3. Name it **Vietnam Companies & Tenders** and paste `https://thecompanyatlas.com/mcp/vietnam`
4. Save and sign in with Google when prompted

