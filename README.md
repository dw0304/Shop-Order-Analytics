# TikTok Shop Order Analytics — Dashboard v2

Streamlit MVP for TikTok Shop All Order exports.

## Business rules
- 1 SKU row = 1 order record.
- Do not deduplicate Order ID.
- `Cancelation/Return Type = Return/Refund` => Refund.
- `Cancelation/Return Type = Cancel` => Cancel.
- Cancel + Tracking ID is a separate `Tracked Cancel` operational bucket; it is **not** counted as Refund.
- GMV = SUM of `SKU Subtotal After Discount` at SKU-line level.
- `Order Amount` is not summed because it can repeat across SKU lines for the same Order ID.
- Net GMV = Gross GMV - Refund GMV - Cancel GMV.
- Refund Type (System/Seller/Buyer) is not inferred unless a reliable source field exists.

## Run on Windows

```bat
pip install -r requirements.txt
streamlit run app.py
```
