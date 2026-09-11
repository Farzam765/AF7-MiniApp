#!/usr/bin/env python3
"""
==============================================================
  AF7_TRADER - SCALP-ONLY AGENT (SMC / ICT / PO3)
  MetaTrader 5 | Fixed Symbol List | Limit-Order Entries
  Fully Automatic Open/Manage/Close | Weekly Win-Rate Report
  ** Uses FREE MetaTrader5 Python Package **
  ** MT5 must be open and logged in **
==============================================================
"""

# ============================================================
# *** SETTINGS - FILL THESE IN BEFORE RUNNING ***
# ============================================================

# --- Telegram ---
TELEGRAM_BOT_TOKEN = "8550167948:AAHO38O-NhzDUU1Yjgaxu6TWTfJv9_2XEaM"
TELEGRAM_CHAT_ID   = "-1003463322431"

# --- MT5 Login (same as your broker login) ---
MT5_LOGIN    = 2002083581
MT5_PASSWORD = "Farzam.Amin.00"
MT5_SERVER   = "JustMarkets-Demo"

# --- Trading Mode ---
# Always fully automatic. No Telegram confirmation is required to open trades.
TRADE_MODE = "AUTO"

# --- Fixed Symbol Universe (base names — broker suffixes are auto-resolved) ---
BASE_SYMBOLS = [
    "EURUSD", "AUDUSD", "XAUUSD", "US100", "US30", "US500",
    "USDCAD", "GBPUSD", "USDJPY", "GBPJPY", "EURAUD", "AUDCAD",
    "AUDCHF", "CHFJPY", "EURCAD", "EURGBP", "EURJPY", "GBPCAD",
    "GBPCHF", "BTCUSD",
]
WEEKEND_ONLY_KEYWORDS = ["BTC"]   # only these trade on Sat/Sun

# --- Lot Size Rules (auto-scales up as balance grows) ---
LOT_MAX = 0.50

# --- Capital Protection ---
# Position size is now based on % of equity risked per trade, not a fixed lot table —
# this is the single biggest lever for protecting other people's money: every trade
# risks the same small slice of the account no matter which symbol it's on.
RISK_PER_TRADE_PCT   = 5.0   # % of balance risked on a single trade's stop loss
MAX_TOTAL_RISK_PCT   = 7.0   # cap on combined risk across every open/pending trade at once

# --- Spread Filter ---
# Reject an entry if the live spread is abnormally wide (news spikes, illiquid hours,
# broker rollover) — a wide spread alone can turn a good setup into an instant loss.
MAX_SPREAD_PIPS = 3
SYMBOL_MAX_SPREAD_OVERRIDE = {
    "XAU": 50, "BTC": 80, "US30": 100, "US100": 50, "US500": 30,
}

# --- Timeframes ---
TREND_TF   = "H1"     # higher-timeframe bias/structure (analysis only)
ZONE_TF    = "M15"    # structure + zone detection: OB / FVG (analysis only, NOT the entry trigger)
CONFIRM_TF = "M1"     # ← THE ACTUAL ENTRY TRIGGER: M1 candle confirms and fires the trade

# --- Hybrid Execution ---
# After M1 confirmation: if price is still close to the zone, enter at MARKET
# immediately (don't risk missing the move). If price has already moved away,
# rest a LIMIT order at the zone instead, in case price comes back.
ENTRY_MAX_SLIPPAGE_PIPS = 5

# --- Analysis Settings ---
MIN_SETUP_SCORE = 8          # tested 9: no real winrate gain (71% vs 72.6%) but trades roughly halved — reverted
SYMBOL_MIN_SCORE_OVERRIDE = {
    "BTC": 10,     # 30d backtest: score 8 → PF 0.46 (loss). score 10 → PF 1.07 (improved).
    "US100": 10,   # 60d backtest: 14 trades, 15.4% WR, PF 0.46 — clear underperformer at score 8.
}
RR_MIN          = 1.0        # fixed 1:1 target per Farzam's request (was RR_MIN=1.0, RR_MAX=5.0)
RR_MAX          = 1.0        # NOTE: this trades "higher winrate" for "lower profit factor" —
                              # winners can no longer run to 2-5R like they did in earlier backtests
OB_MIN_MOVE_PCT = 0.3

# --- Scan / Watchlist Settings ---
SCAN_INTERVAL_MINUTES    = 1
MONITOR_INTERVAL_SECONDS = 20
MAX_ACTIVE_TRADES        = 8
MAX_WATCH_MINUTES        = 90     # drop a candidate if it never confirms within this long
PENDING_ORDER_EXPIRY_MIN = 90     # cancel unfilled limit order after this long

# --- Correlation Lock ---
# Blocks a new trade if it would push same-direction exposure on a shared
# currency (e.g. EUR via EURUSD + EURGBP + EURJPY all long EUR at once) too high.
MAX_CURRENCY_EXPOSURE = 2

# --- Break Even ---
BE_TRIGGER_RR  = 1.0
BE_BUFFER_PIPS = 2

# --- Risk Guard (circuit breaker) ---
MAX_CONSECUTIVE_LOSSES = 3     # pause new entries after this many losses in a row
MAX_DAILY_LOSS_PCT     = 5.0   # pause for the rest of the day past this drawdown
MAX_WEEKLY_LOSS_PCT    = 10.0  # pause for the rest of the week past this drawdown

# --- News Filter ---
NEWS_BLOCK_BEFORE_MIN = 60   # no new signals/entries this long before high/med news
NEWS_BLOCK_AFTER_MIN  = 60   # ...and this long after
SYMBOL_NEWS_MAP = {
    "US100": ["USD"], "US30": ["USD"], "US500": ["USD"],
    "XAUUSD": ["USD"], "BTCUSD": [],   # crypto: no forex-calendar blackout
}

# --- Magic Number ---
MAGIC_NUMBER = 20260906

# --- Status Server (feeds the mini app) ---
# Serves current settings + live signals + trade history as JSON at /status.
# The mini app polls this URL — see STATUS_URL at the top of the HTML file.
# NOTE: Telegram Mini Apps require a public HTTPS URL. For testing, tunnel this
# local port with something like ngrok (`ngrok http 8787`) and put the https://
# address it gives you into the mini app's STATUS_URL. For real use you'll want
# it running behind a real domain + SSL certificate.
STATUS_SERVER_PORT = 8787

# ============================================================

import logging
import os
import time
import uuid
import json
import http.server
import socketserver
import random
import schedule
import threading
import requests
from datetime import datetime, timedelta
from typing import Dict, List, Optional, Tuple
from dataclasses import dataclass
from enum import Enum

try:
    import MetaTrader5 as mt5
except ImportError:
    print("ERROR: MetaTrader5 not installed!")
    print("Run: pip install MetaTrader5")
    exit(1)

# ─────────────────────────────────────────────
# LOGGING
# ─────────────────────────────────────────────
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
    handlers=[
        logging.FileHandler("af7_agent.log", encoding="utf-8"),
        logging.StreamHandler(),
    ],
)
log = logging.getLogger(__name__)


# ─────────────────────────────────────────────
# ENUMS & DATA CLASSES
# ─────────────────────────────────────────────
class Direction(str, Enum):
    LONG  = "LONG"
    SHORT = "SHORT"

class TradeStatus(str, Enum):
    WATCHING = "WATCHING"   # candidate zone found, waiting for confirmation
    PENDING  = "PENDING"    # limit order placed on broker, waiting for fill
    OPEN     = "OPEN"
    CLOSED   = "CLOSED"
    EXPIRED  = "EXPIRED"


TF_MAP = {
    "M1":  None, "M5":  None, "M15": None, "M30": None,
    "H1":  None, "H4":  None, "D1":  None, "W1":  None,
}
# filled in after mt5 is available (constants require the module)
TF_MAP = {
    "M1":  mt5.TIMEFRAME_M1,  "M5":  mt5.TIMEFRAME_M5,
    "M15": mt5.TIMEFRAME_M15, "M30": mt5.TIMEFRAME_M30,
    "H1":  mt5.TIMEFRAME_H1,  "H4":  mt5.TIMEFRAME_H4,
    "D1":  mt5.TIMEFRAME_D1,  "W1":  mt5.TIMEFRAME_W1,
}


@dataclass
class Signal:
    id: str
    symbol: str
    direction: Direction
    timeframe: str
    entry: float
    sl: float
    tp1: float
    tp2: float
    tp3: float
    lot_size: float
    rr1: float
    rr2: float
    rr3: float
    score: int
    timestamp: datetime
    status: TradeStatus  = TradeStatus.PENDING
    order_ticket: int    = 0     # broker pending-order / position ticket
    tp1_hit: bool        = False
    tp2_hit: bool        = False
    tp3_hit: bool        = False
    be_moved: bool       = False
    active: bool         = True
    pip_value: float     = 0.0001
    placed_at: datetime  = None


@dataclass
class AccountInfo:
    balance: float     = 0.0
    equity: float      = 0.0
    free_margin: float = 0.0
    currency: str      = "USD"


# ─────────────────────────────────────────────
# PURE PYTHON MATH (no numpy)
# ─────────────────────────────────────────────
def _mean(v: list) -> float:
    return sum(v) / len(v) if v else 0.0

def _ema(prices: list, period: int) -> list:
    if not prices or len(prices) < period:
        return [prices[-1]] if prices else [0.0]
    k   = 2.0 / (period + 1)
    ema = [_mean(prices[:period])]
    for p in prices[period:]:
        ema.append(p * k + ema[-1] * (1 - k))
    return ema

def _rsi(prices: list, period: int = 14) -> float:
    if len(prices) < period + 1:
        return 50.0
    gains, losses = [], []
    for i in range(1, len(prices)):
        d = prices[i] - prices[i - 1]
        gains.append(max(d, 0))
        losses.append(max(-d, 0))
    ag = _mean(gains[-period:])
    al = _mean(losses[-period:])
    return 100.0 if al == 0 else 100 - (100 / (1 + ag / al))

def _macd_hist(prices: list) -> Tuple[float, float]:
    if len(prices) < 27:
        return 0.0, 0.0
    fast = _ema(prices, 12)
    slow = _ema(prices, 26)
    n    = min(len(fast), len(slow))
    ml   = [fast[-(n - i)] - slow[-(n - i)] for i in range(n)]
    sig  = _ema(ml, 9)
    hc   = ml[-1] - sig[-1] if sig else 0.0
    hp   = ml[-2] - sig[-2] if len(ml) > 1 and len(sig) > 1 else 0.0
    return hc, hp

def _atr(highs: list, lows: list, closes: list, period: int = 14) -> float:
    if len(closes) < period + 1:
        return 0.0
    trs = [max(highs[i] - lows[i], abs(highs[i] - closes[i - 1]),
               abs(lows[i] - closes[i - 1])) for i in range(1, len(closes))]
    return _mean(trs[-period:])

def _vol_avg(vols: list, period: int = 20) -> float:
    return _mean(vols[-period:]) if len(vols) >= period else _mean(vols)

def _pip(symbol: str) -> float:
    s = symbol.upper()
    if "JPY" in s: return 0.01
    if "XAU" in s: return 0.1
    if "XAG" in s: return 0.01
    if any(k in s for k in ("NAS100", "US30", "US500")): return 1.0
    if "BTC" in s: return 1.0
    return 0.0001

def currencies_for(symbol: str) -> List[str]:
    s = symbol.upper()
    for key, curs in SYMBOL_NEWS_MAP.items():
        if s.startswith(key):
            return curs
    if len(s) >= 6:
        return [s[:3], s[3:6]]
    return []

def is_weekend() -> bool:
    return datetime.utcnow().weekday() >= 5  # 5=Sat, 6=Sun

def min_score_for(symbol: str) -> int:
    s = symbol.upper()
    for key, val in SYMBOL_MIN_SCORE_OVERRIDE.items():
        if key in s:
            return val
    return MIN_SETUP_SCORE

def max_spread_for(symbol: str) -> float:
    s = symbol.upper()
    for key, val in SYMBOL_MAX_SPREAD_OVERRIDE.items():
        if key in s:
            return val
    return MAX_SPREAD_PIPS

def currency_exposure(symbol: str, direction: str) -> Dict[str, int]:
    """Returns {currency: +1/-1} for correlation-lock accounting.
    Indices are equity risk, not FX risk, so they're excluded (empty dict)."""
    s = symbol.upper()
    if any(k in s for k in ("US30", "US100", "US500", "NAS100")):
        return {}
    if "XAU" in s:
        base, quote = "XAU", "USD"
    elif "BTC" in s:
        base, quote = "BTC", "USD"
    elif len(s) >= 6:
        base, quote = s[:3], s[3:6]
    else:
        return {}
    sign = 1 if direction == "LONG" else -1
    return {base: sign, quote: -sign}


# ─────────────────────────────────────────────
# MT5 CONNECTOR
# ─────────────────────────────────────────────
class MT5Connector:

    def __init__(self):
        self.connected = False
        self.symbol_map: Dict[str, str] = {}   # base -> broker symbol name

    def connect(self) -> bool:
        if not mt5.initialize():
            log.error(f"MT5 initialize failed: {mt5.last_error()}")
            return False
        if MT5_LOGIN and MT5_PASSWORD and MT5_SERVER:
            if not mt5.login(login=MT5_LOGIN, password=MT5_PASSWORD, server=MT5_SERVER):
                log.error(f"MT5 login failed: {mt5.last_error()}")
                return False
        info = mt5.account_info()
        if info is None:
            log.error("Cannot get account info")
            return False
        self.connected = True
        log.info(f"MT5 connected | Account: {info.login} | Balance: {info.balance} {info.currency}")
        return True

    def disconnect(self):
        mt5.shutdown()
        self.connected = False

    def get_account(self) -> Optional[AccountInfo]:
        info = mt5.account_info()
        if info is None:
            return None
        return AccountInfo(balance=info.balance, equity=info.equity,
                            free_margin=info.margin_free, currency=info.currency)

    def resolve_symbols(self) -> Dict[str, str]:
        """Match our fixed base names to whatever this broker actually calls them
        (suffixes/prefixes like .m, .r, -ECN, m_ etc.)."""
        all_syms = mt5.symbols_get()
        names = [s.name for s in all_syms] if all_syms else []
        mapping = {}
        for base in BASE_SYMBOLS:
            candidates = [n for n in names if base in n.upper()]
            if not candidates:
                log.warning(f"Symbol not found on broker: {base}")
                continue
            # prefer exact match, else shortest (least suffix noise)
            exact = [n for n in candidates if n.upper() == base]
            chosen = exact[0] if exact else min(candidates, key=len)
            info = mt5.symbol_info(chosen)
            if info and not info.visible:
                mt5.symbol_select(chosen, True)
            mapping[base] = chosen
        self.symbol_map = mapping
        return mapping

    def get_candles(self, symbol: str, tf: str, count: int = 200) -> dict:
        mt5_tf = TF_MAP.get(tf)
        if mt5_tf is None:
            return {}
        rates = mt5.copy_rates_from_pos(symbol, mt5_tf, 0, count)
        if rates is None or len(rates) == 0:
            return {}
        return {
            "open":   [r["open"] for r in rates],
            "high":   [r["high"] for r in rates],
            "low":    [r["low"] for r in rates],
            "close":  [r["close"] for r in rates],
            "volume": [float(r["tick_volume"]) for r in rates],
        }

    def get_price(self, symbol: str) -> Tuple[float, float]:
        tick = mt5.symbol_info_tick(symbol)
        if tick is None:
            return 0.0, 0.0
        return tick.bid, tick.ask

    @staticmethod
    def _filling_type(info):
        """Picks a filling mode the symbol actually supports instead of assuming IOC —
        many brokers reject IOC on certain symbols (retcode 10030, 'Unsupported filling
        mode'), which is the most common reason orders silently fail to open."""
        mode = getattr(info, "filling_mode", 0) or 0
        if mode & 2:
            return mt5.ORDER_FILLING_IOC
        if mode & 1:
            return mt5.ORDER_FILLING_FOK
        return mt5.ORDER_FILLING_RETURN

    def _send_with_fallback(self, request: dict, label: str):
        """Sends the order; if it fails specifically on filling mode, retries with the
        other supported modes before giving up. Always logs the broker's own retcode
        and comment so the real reason is visible in af7_agent.log."""
        candidates = [request["type_filling"], mt5.ORDER_FILLING_IOC,
                      mt5.ORDER_FILLING_FOK, mt5.ORDER_FILLING_RETURN]
        tried = []
        result = None
        for fm in candidates:
            if fm in tried:
                continue
            tried.append(fm)
            request["type_filling"] = fm
            result = mt5.order_send(request)
            if result is not None and result.retcode == mt5.TRADE_RETCODE_DONE:
                return result
            if result is None:
                log.error(f"{label} {request.get('symbol')}: order_send returned None — "
                          f"mt5.last_error()={mt5.last_error()}")
                return None
            if result.retcode != 10030:  # not a filling-mode problem — retrying won't help
                break
        if result is not None:
            log.error(f"{label} {request.get('symbol')} failed after {len(tried)} filling mode(s) "
                      f"tried={tried}: retcode={result.retcode} comment='{result.comment}'")
        return result

    def place_limit_order(self, symbol: str, direction: str, lot: float,
                           price: float, sl: float, tp: float,
                           expiry_minutes: int) -> int:
        info = mt5.symbol_info(symbol)
        if info is None:
            log.error(f"Symbol {symbol} not found")
            return 0
        if not info.visible:
            mt5.symbol_select(symbol, True)

        order_t = mt5.ORDER_TYPE_BUY_LIMIT if direction == "LONG" else mt5.ORDER_TYPE_SELL_LIMIT
        expiration = datetime.utcnow() + timedelta(minutes=expiry_minutes)

        request = {
            "action":       mt5.TRADE_ACTION_PENDING,
            "symbol":       symbol,
            "volume":       lot,
            "type":         order_t,
            "price":        round(price, info.digits),
            "sl":           round(sl, info.digits),
            "tp":           round(tp, info.digits),
            "magic":        MAGIC_NUMBER,
            "comment":      "AF7_Scalp",
            "type_time":    mt5.ORDER_TIME_SPECIFIED,
            "expiration":   expiration,
            "type_filling": self._filling_type(info),
        }
        result = self._send_with_fallback(request, "Limit order")
        if result is None or result.retcode != mt5.TRADE_RETCODE_DONE:
            return 0
        log.info(f"Limit order placed: {symbol} {direction} lot={lot} @ {price} ticket={result.order}")
        return result.order

    def place_market_order(self, symbol: str, direction: str, lot: float,
                            sl: float, tp: float) -> int:
        info = mt5.symbol_info(symbol)
        if info is None:
            log.error(f"Symbol {symbol} not found")
            return 0
        if not info.visible:
            mt5.symbol_select(symbol, True)
        tick = mt5.symbol_info_tick(symbol)
        if tick is None:
            return 0
        order_t = mt5.ORDER_TYPE_BUY if direction == "LONG" else mt5.ORDER_TYPE_SELL
        price = tick.ask if direction == "LONG" else tick.bid

        request = {
            "action":       mt5.TRADE_ACTION_DEAL,
            "symbol":       symbol,
            "volume":       lot,
            "type":         order_t,
            "price":        round(price, info.digits),
            "sl":           round(sl, info.digits),
            "tp":           round(tp, info.digits),
            "magic":        MAGIC_NUMBER,
            "comment":      "AF7_Scalp_Mkt",
            "type_time":    mt5.ORDER_TIME_GTC,
            "type_filling": self._filling_type(info),
        }
        result = self._send_with_fallback(request, "Market order")
        if result is None or result.retcode != mt5.TRADE_RETCODE_DONE:
            return 0
        log.info(f"Market order filled: {symbol} {direction} lot={lot} @ {price} ticket={result.order}")
        return result.order

    def cancel_order(self, ticket: int) -> bool:
        request = {"action": mt5.TRADE_ACTION_REMOVE, "order": ticket}
        result = mt5.order_send(request)
        return result is not None and result.retcode == mt5.TRADE_RETCODE_DONE

    def get_pending_tickets(self) -> set:
        orders = mt5.orders_get()
        if not orders:
            return set()
        return {o.ticket for o in orders if o.magic == MAGIC_NUMBER}

    def get_open_tickets(self) -> set:
        pos = mt5.positions_get()
        if not pos:
            return set()
        return {p.ticket for p in pos if p.magic == MAGIC_NUMBER}

    def modify_sl(self, ticket: int, new_sl: float) -> bool:
        pos = mt5.positions_get(ticket=ticket)
        if not pos:
            return False
        p = pos[0]
        request = {"action": mt5.TRADE_ACTION_SLTP, "position": ticket,
                   "sl": new_sl, "tp": p.tp}
        result = mt5.order_send(request)
        return result is not None and result.retcode == mt5.TRADE_RETCODE_DONE

    def close_partial(self, ticket: int, lot: float, symbol: str, direction: str) -> bool:
        info = mt5.symbol_info(symbol)
        tick = mt5.symbol_info_tick(symbol)
        price = tick.bid if direction == "LONG" else tick.ask
        order_t = mt5.ORDER_TYPE_SELL if direction == "LONG" else mt5.ORDER_TYPE_BUY
        request = {
            "action": mt5.TRADE_ACTION_DEAL, "symbol": symbol, "volume": lot,
            "type": order_t, "price": price, "position": ticket,
            "magic": MAGIC_NUMBER, "comment": "AF7_Partial",
            "type_time": mt5.ORDER_TIME_GTC,
            "type_filling": self._filling_type(info) if info else mt5.ORDER_FILLING_IOC,
        }
        result = self._send_with_fallback(request, "Partial close")
        return result is not None and result.retcode == mt5.TRADE_RETCODE_DONE

    def close_trade(self, ticket: int, symbol: str, direction: str) -> bool:
        pos = mt5.positions_get(ticket=ticket)
        if not pos:
            return False
        return self.close_partial(ticket, pos[0].volume, symbol, direction)


# ─────────────────────────────────────────────
# LOT CALCULATOR
# Primary method: risk exactly RISK_PER_TRADE_PCT of the account on this trade's
# stop distance, using the broker's real tick value (correct for every instrument
# class — FX, gold, indices, crypto — without hardcoding pip worth per symbol).
# Falls back to a simple balance-tiered table only if broker data isn't available.
# ─────────────────────────────────────────────
class LotCalculator:
    @staticmethod
    def calculate(symbol: str, balance: float, risk_distance: float = None) -> float:
        info = mt5.symbol_info(symbol) if hasattr(mt5, "symbol_info") else None
        if info and risk_distance and risk_distance > 0:
            tick_value = getattr(info, "trade_tick_value", 0) or 0
            tick_size = getattr(info, "trade_tick_size", 0) or getattr(info, "point", 0) or 0
            vol_min = getattr(info, "volume_min", 0.01) or 0.01
            vol_max = getattr(info, "volume_max", LOT_MAX) or LOT_MAX
            vol_step = getattr(info, "volume_step", 0.01) or 0.01
            if tick_value > 0 and tick_size > 0:
                money_per_unit_per_lot = tick_value / tick_size
                money_at_risk_per_lot = risk_distance * money_per_unit_per_lot
                if money_at_risk_per_lot > 0:
                    risk_amount = balance * (RISK_PER_TRADE_PCT / 100)
                    lot = risk_amount / money_at_risk_per_lot
                    lot = round(lot / vol_step) * vol_step
                    lot = max(vol_min, min(lot, vol_max, LOT_MAX))
                    return round(lot, 2)

        # Fallback: balance-tiered table (used only if broker tick data is missing)
        s = symbol.upper()
        if any(k in s for k in ("NAS100", "US30", "US100", "US500")):
            base, step_bal, step_lot = 0.10, 500, 0.05
        elif "XAU" in s:
            base, step_bal, step_lot = 0.01, 300, 0.01
        elif "BTC" in s:
            base, step_bal, step_lot = 0.01, 500, 0.01
        else:
            base, step_bal, step_lot = 0.01, 100, 0.01
        tiers = max(0, int((balance - 50) // step_bal)) if balance > 50 else 0
        lot = base + tiers * step_lot
        return min(round(lot, 2), LOT_MAX)


# ─────────────────────────────────────────────
# SMC / ICT ANALYZER
# ─────────────────────────────────────────────
class SMCAnalyzer:

    @staticmethod
    def swing_highs(highs: list, left=5, right=3) -> List[dict]:
        out = []
        for i in range(left, len(highs) - right):
            w = highs[i - left: i + right + 1]
            if highs[i] == max(w):
                out.append({"idx": i, "price": highs[i]})
        return out

    @staticmethod
    def swing_lows(lows: list, left=5, right=3) -> List[dict]:
        out = []
        for i in range(left, len(lows) - right):
            w = lows[i - left: i + right + 1]
            if lows[i] == min(w):
                out.append({"idx": i, "price": lows[i]})
        return out

    @staticmethod
    def market_structure(data: dict) -> dict:
        H = data.get("high", []); L = data.get("low", []); C = data.get("close", [])
        sh = SMCAnalyzer.swing_highs(H); sl = SMCAnalyzer.swing_lows(L)
        empty = {"trend": "RANGING", "bos": False, "choch": False, "swing_highs": sh, "swing_lows": sl}
        if len(sh) < 2 or len(sl) < 2:
            return empty
        hh = sh[-1]["price"] > sh[-2]["price"]; hl = sl[-1]["price"] > sl[-2]["price"]
        lh = sh[-1]["price"] < sh[-2]["price"]; ll = sl[-1]["price"] < sl[-2]["price"]
        trend = "BULLISH" if (hh and hl) else ("BEARISH" if (lh and ll) else "RANGING")
        lc = C[-1] if C else 0
        bb = lc > sh[-1]["price"]; bear = lc < sl[-1]["price"]
        bos = bb or bear
        choch = (trend == "BULLISH" and bear) or (trend == "BEARISH" and bb)
        return {"trend": trend, "bos": bos, "choch": choch, "swing_highs": sh, "swing_lows": sl}

    @staticmethod
    def order_blocks(data: dict, lookback=60) -> List[dict]:
        O = data["open"]; H = data["high"]; L = data["low"]; C = data["close"]
        obs = []; start = max(1, len(C) - lookback)
        for i in range(start, len(C) - 4):
            if C[i] < O[i]:
                fh = max(H[i + 1:i + 6]) if i + 6 <= len(H) else H[i]
                mp = (fh - L[i]) / (L[i] + 1e-10) * 100
                if mp >= OB_MIN_MOVE_PCT:
                    obs.append({"type": "BULLISH", "top": O[i], "bottom": L[i],
                                "mid": (O[i] + L[i]) / 2, "strength": mp})
            elif C[i] > O[i]:
                fl = min(L[i + 1:i + 6]) if i + 6 <= len(L) else L[i]
                mp = (H[i] - fl) / (H[i] + 1e-10) * 100
                if mp >= OB_MIN_MOVE_PCT:
                    obs.append({"type": "BEARISH", "top": H[i], "bottom": C[i],
                                "mid": (H[i] + C[i]) / 2, "strength": mp})
        obs.sort(key=lambda x: x["strength"], reverse=True)
        return obs[:8]

    @staticmethod
    def fair_value_gaps(data: dict, lookback=60) -> List[dict]:
        H = data["high"]; L = data["low"]
        fvgs = []; start = max(2, len(H) - lookback)
        for i in range(start, len(H)):
            if H[i - 2] < L[i]:
                fvgs.append({"type": "BULLISH", "top": L[i], "bottom": H[i - 2], "mid": (L[i] + H[i - 2]) / 2})
            elif L[i - 2] > H[i]:
                fvgs.append({"type": "BEARISH", "top": L[i - 2], "bottom": H[i], "mid": (L[i - 2] + H[i]) / 2})
        return fvgs

    @staticmethod
    def liquidity_sweep(data: dict, sh_list, sl_list) -> Optional[dict]:
        H = data["high"]; L = data["low"]; C = data["close"]
        if len(C) < 10:
            return None
        rh = H[-10:]; rl = L[-10:]; lc = C[-1]
        for sh in sh_list[-6:]:
            if max(rh) > sh["price"] and lc < sh["price"]:
                return {"type": "BEARISH_SWEEP", "level": sh["price"]}
        for sl in sl_list[-6:]:
            if min(rl) < sl["price"] and lc > sl["price"]:
                return {"type": "BULLISH_SWEEP", "level": sl["price"]}
        return None

    @staticmethod
    def in_kill_zone() -> Tuple[bool, str]:
        t = datetime.utcnow().hour * 60 + datetime.utcnow().minute
        if 420 <= t <= 540: return True, "London Open Kill Zone"
        if 780 <= t <= 900: return True, "New York Open Kill Zone"
        if 0 <= t <= 180: return True, "Asian Session Kill Zone"
        if 720 <= t <= 960: return True, "London/NY Overlap"
        return False, ""

    @staticmethod
    def confirms_entry(m5: dict, direction: str, zone_low: float, zone_high: float) -> bool:
        """Lower-timeframe (M5) confirmation: last closed candle must react from the zone."""
        C = m5.get("close", []); O = m5.get("open", [])
        H = m5.get("high", []); L = m5.get("low", [])
        if len(C) < 3:
            return False
        o, c, h, l = O[-2], C[-2], H[-2], L[-2]   # last fully closed candle
        touched_zone = (l <= zone_high) and (h >= zone_low)
        if not touched_zone:
            return False
        body = abs(c - o)
        if direction == "LONG":
            bullish = c > o
            lower_wick_rejection = (min(o, c) - l) > body * 0.6
            return bullish or lower_wick_rejection
        else:
            bearish = c < o
            upper_wick_rejection = (h - max(o, c)) > body * 0.6
            return bearish or upper_wick_rejection


# ─────────────────────────────────────────────
# PO3 (POWER OF THREE) ANALYZER
# Accumulation (Asia) -> Manipulation (London sweep) -> Distribution (NY expansion)
# ─────────────────────────────────────────────
class PO3Analyzer:
    @staticmethod
    def session_phase() -> str:
        h = datetime.utcnow().hour
        if 0 <= h < 7:   return "ACCUMULATION"
        if 7 <= h < 12:  return "MANIPULATION"
        if 12 <= h < 20: return "DISTRIBUTION"
        return "OFF"

    @staticmethod
    def bias(data: dict) -> Optional[str]:
        """Uses M15 data: ~24 candles = 6h Asian range, then checks if the more
        recent candles swept that range and reversed (classic PO3 manipulation)."""
        H = data.get("high", []); L = data.get("low", []); C = data.get("close", [])
        if len(H) < 40:
            return None
        asia_h = max(H[-40:-16]); asia_l = min(L[-40:-16])
        recent_h = max(H[-16:]); recent_l = min(L[-16:])
        last_close = C[-1]
        if recent_h > asia_h and last_close < asia_h:
            return "BEARISH_PO3"
        if recent_l < asia_l and last_close > asia_l:
            return "BULLISH_PO3"
        return None


# ─────────────────────────────────────────────
# NEWS / ECONOMIC CALENDAR
# ─────────────────────────────────────────────
class ForexCalendar:
    def __init__(self):
        self._cache = []
        self._cache_time = None

    def _events(self) -> list:
        if self._cache_time and (datetime.utcnow() - self._cache_time).total_seconds() < 900:
            return self._cache
        try:
            resp = requests.get("https://nfs.faireconomy.media/ff_calendar_thisweek.json", timeout=10)
            self._cache = resp.json() if resp.status_code == 200 else []
        except Exception as e:
            log.error(f"Calendar fetch error: {e}")
            self._cache = []
        self._cache_time = datetime.utcnow()
        return self._cache

    def is_blackout(self, symbol: str) -> Optional[dict]:
        curs = currencies_for(symbol)
        if not curs:
            return None
        now = datetime.utcnow()
        for ev in self._events():
            if ev.get("currency", "") not in curs:
                continue
            if ev.get("impact", "") not in ("High", "Medium"):
                continue
            try:
                dt = datetime.strptime(ev["date"], "%Y-%m-%dT%H:%M:%S%z").replace(tzinfo=None)
            except Exception:
                continue
            diff_min = (dt - now).total_seconds() / 60.0
            if -NEWS_BLOCK_AFTER_MIN <= diff_min <= NEWS_BLOCK_BEFORE_MIN:
                return {"title": ev.get("title", ""), "currency": ev.get("currency", ""),
                        "impact": ev.get("impact", ""), "minutes_to_event": round(diff_min, 1)}
        return None


# ─────────────────────────────────────────────
# TELEGRAM BOT — sends signal parameters only, no analysis text
# ─────────────────────────────────────────────
class TelegramBot:
    def __init__(self):
        self.base = f"https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}"
        self.chat_id = TELEGRAM_CHAT_ID

    def send(self, text: str) -> Optional[int]:
        try:
            r = requests.post(f"{self.base}/sendMessage",
                               json={"chat_id": self.chat_id, "text": text, "parse_mode": "HTML"},
                               timeout=15)
            if r.status_code == 200:
                return r.json().get("result", {}).get("message_id")
            return None
        except Exception as e:
            log.error(f"Telegram error: {e}")
            return None

    def send_signal(self, sig: Signal) -> bool:
        de = "🟢" if sig.direction == Direction.LONG else "🔴"
        is_market = sig.status == TradeStatus.OPEN
        dl = ("LONG ▲ MARKET BUY" if is_market else "LONG ▲ BUY LIMIT") if sig.direction == Direction.LONG else \
             ("SHORT ▼ MARKET SELL" if is_market else "SHORT ▼ SELL LIMIT")
        footer = "🚀 <i>Filled at market.</i>" if is_market else "⏳ <i>Resting as a limit order. Executes automatically.</i>"
        return self.send(
            f"{'─'*32}\n⚡ <b>SCALP SIGNAL</b>\n{'─'*32}\n\n"
            f"{de} <b>{sig.symbol}</b>  |  {dl}\n"
            f"🆔 <code>{sig.id}</code>\n\n"
            f"   🔸 Entry          →  <code>{sig.entry:.5f}</code>\n"
            f"   🛑 SL            →  <code>{sig.sl:.5f}</code>\n"
            f"   🥇 TP1 ({sig.rr1:.1f}R)   →  <code>{sig.tp1:.5f}</code>\n"
            f"   🥈 TP2 ({sig.rr2:.1f}R)   →  <code>{sig.tp2:.5f}</code>\n"
            f"   🥉 TP3 ({sig.rr3:.1f}R)   →  <code>{sig.tp3:.5f}</code>\n"
            f"   📦 Lot           →  {sig.lot_size}\n\n"
            f"{footer}"
        ) is not None

    def send_tp_hit(self, sig: Signal, n: int) -> bool:
        return self.send(f"✅ <b>{sig.symbol}</b> TP{n} hit — <code>{sig.id}</code>") is not None

    def send_sl_hit(self, sig: Signal, be: bool) -> bool:
        label = "Break-even stop-out" if be else "STOP LOSS hit"
        return self.send(f"🛑 <b>{sig.symbol}</b> {label} — <code>{sig.id}</code>") is not None

    def send_be_moved(self, sig: Signal) -> bool:
        return self.send(f"🛡 <b>{sig.symbol}</b> moved to break-even — <code>{sig.id}</code>") is not None

    def send_trade_opened(self, sig: Signal) -> bool:
        return self.send(f"🚀 <b>{sig.symbol}</b> limit filled, trade is live — <code>{sig.id}</code>") is not None

    def send_full_tp(self, sig: Signal) -> bool:
        return self.send(f"🏁 <b>{sig.symbol}</b> TP3 hit, position closed — <code>{sig.id}</code>") is not None

    def send_expired(self, sig: Signal) -> bool:
        return self.send(f"⌛ <b>{sig.symbol}</b> limit order expired unfilled — <code>{sig.id}</code>") is not None

    def send_news_cancel(self, sig: Signal, ev: dict) -> bool:
        return self.send(
            f"📰 <b>{sig.symbol}</b> pending order cancelled — {ev['currency']} "
            f"{ev['impact']} news: {ev['title'][:60]}"
        ) is not None

    def send_weekly_report(self, stats: dict) -> bool:
        if not stats:
            return self.send("📊 <b>Weekly Report</b>\nNo closed trades in the last 7 days.") is not None
        return self.send(
            f"{'─'*32}\n📊 <b>WEEKLY REPORT</b>\n{'─'*32}\n\n"
            f"   Trades   → {stats['total']}\n"
            f"   Wins     → {stats['wins']}\n"
            f"   Losses   → {stats['losses']}\n"
            f"   Break-even → {stats.get('be', 0)}\n"
            f"   Win rate → {stats['winrate']}%\n"
            f"   Avg RR   → {stats['avg_rr']}"
        ) is not None


# ─────────────────────────────────────────────
# TRADE HISTORY (for weekly win-rate reporting)
# ─────────────────────────────────────────────
class TradeHistory:
    FILE = "af7_trade_history.jsonl"

    def record(self, symbol: str, result: str, rr: float):
        entry = {"time": datetime.utcnow().isoformat(), "symbol": symbol, "result": result, "rr": rr}
        try:
            with open(self.FILE, "a", encoding="utf-8") as f:
                f.write(json.dumps(entry) + "\n")
        except Exception as e:
            log.error(f"TradeHistory write error: {e}")

    def weekly_stats(self) -> Optional[dict]:
        if not os.path.exists(self.FILE):
            return None
        cutoff = datetime.utcnow() - timedelta(days=7)
        wins = losses = be = 0
        rr_sum = 0.0
        try:
            with open(self.FILE, encoding="utf-8") as f:
                for line in f:
                    try:
                        e = json.loads(line)
                        t = datetime.fromisoformat(e["time"])
                        if t < cutoff:
                            continue
                        if e["result"] == "WIN": wins += 1
                        elif e["result"] == "LOSS": losses += 1
                        elif e["result"] == "BE": be += 1
                        rr_sum += e.get("rr", 0)
                    except Exception:
                        continue
        except Exception as e:
            log.error(f"TradeHistory read error: {e}")
            return None
        total = wins + losses
        if total + be == 0:
            return None
        return {"wins": wins, "losses": losses, "be": be, "total": total,
                "winrate": round(wins / total * 100, 1) if total else 0.0,
                "avg_rr": round(rr_sum / (total + be), 2) if (total + be) else 0.0}


# ─────────────────────────────────────────────
# RISK GUARD — pauses new entries on consecutive losses or drawdown limits.
# Never touches already-open trades; those are still managed to SL/TP as normal.
# ─────────────────────────────────────────────
class RiskGuard:
    STATE_FILE = "af7_risk_state.json"

    def __init__(self):
        self.state = self._load()

    def _load(self) -> dict:
        default = {
            "date": None, "week": None,
            "daily_start_balance": None, "weekly_start_balance": None,
            "consecutive_losses": 0,
        }
        if os.path.exists(self.STATE_FILE):
            try:
                with open(self.STATE_FILE, encoding="utf-8") as f:
                    default.update(json.load(f))
            except Exception:
                pass
        return default

    def _save(self):
        try:
            with open(self.STATE_FILE, "w", encoding="utf-8") as f:
                json.dump(self.state, f)
        except Exception as e:
            log.error(f"RiskGuard save error: {e}")

    def _roll_periods(self, balance: float):
        today = datetime.utcnow().strftime("%Y-%m-%d")
        week = datetime.utcnow().strftime("%Y-W%W")
        changed = False
        if self.state["date"] != today:
            self.state["date"] = today
            self.state["daily_start_balance"] = balance
            changed = True
        if self.state["week"] != week:
            self.state["week"] = week
            self.state["weekly_start_balance"] = balance
            self.state["consecutive_losses"] = 0
            changed = True
        if changed:
            self._save()

    def on_trade_closed(self, result: str):
        if result == "LOSS":
            self.state["consecutive_losses"] += 1
        elif result == "WIN":
            self.state["consecutive_losses"] = 0
        self._save()

    def check(self, balance: float) -> Optional[str]:
        """Returns a halt reason, or None if new entries are allowed."""
        self._roll_periods(balance)
        if self.state["consecutive_losses"] >= MAX_CONSECUTIVE_LOSSES:
            return f"{MAX_CONSECUTIVE_LOSSES} consecutive losses — resumes tomorrow"
        dsb = self.state.get("daily_start_balance") or balance
        wsb = self.state.get("weekly_start_balance") or balance
        daily_dd = (dsb - balance) / dsb * 100 if dsb else 0
        weekly_dd = (wsb - balance) / wsb * 100 if wsb else 0
        if daily_dd >= MAX_DAILY_LOSS_PCT:
            return f"daily drawdown {daily_dd:.1f}% (limit {MAX_DAILY_LOSS_PCT}%) — resumes tomorrow"
        if weekly_dd >= MAX_WEEKLY_LOSS_PCT:
            return f"weekly drawdown {weekly_dd:.1f}% (limit {MAX_WEEKLY_LOSS_PCT}%) — resumes next week"
        return None


# ─────────────────────────────────────────────
# SIGNAL GENERATOR
# ─────────────────────────────────────────────
class SignalGenerator:
    def __init__(self, conn: MT5Connector):
        self.conn = conn
        self.smc = SMCAnalyzer()
        self.po3 = PO3Analyzer()
        self.lc = LotCalculator()

    def _score(self, direction, ltf_t, bos, choch, has_ob, has_fvg, sweep,
               rsi, macd_bull, vol_ok, kz, htf_ok, po3_aligned):
        s, r = 0, []
        if htf_ok:                                       s += 2; r.append("HTF trend aligned")
        if (direction == "LONG" and ltf_t == "BULLISH") or \
           (direction == "SHORT" and ltf_t == "BEARISH"): s += 1; r.append("Structure confirmed")
        if bos:                                           s += 1; r.append("BOS")
        if choch:                                         s += 1; r.append("CHoCH")
        if has_ob:                                        s += 2; r.append("Order Block")
        if has_fvg:                                       s += 1; r.append("FVG")
        if sweep:                                         s += 1; r.append("Liquidity sweep")
        if (direction == "LONG" and 30 < rsi < 60) or \
           (direction == "SHORT" and 40 < rsi < 70):       s += 1; r.append("RSI valid")
        if macd_bull and direction == "LONG":              s += 1; r.append("MACD bullish")
        if (not macd_bull) and direction == "SHORT":       s += 1; r.append("MACD bearish")
        if vol_ok:                                         s += 1; r.append("Volume surge")
        if kz:                                             s += 1; r.append("Kill zone")
        if po3_aligned:                                    s += 1; r.append("PO3 aligned")
        return min(s, 12), r

    def find_zone(self, symbol: str, direction: str) -> Optional[dict]:
        """Live path: fetches current candles then delegates to the pure analysis function."""
        dL = self.conn.get_candles(symbol, ZONE_TF, 200)
        dH = self.conn.get_candles(symbol, TREND_TF, 200)
        if not dL or not dH or len(dL.get("close", [])) < 50:
            return None
        return self._analyze_data(symbol, direction, dL, dH)

    def _analyze_data(self, symbol: str, direction: str, dL: dict, dH: dict) -> Optional[dict]:
        """Pure analysis on already-fetched candle data. Used live and by the Backtester
        (same code path both places, so backtest results reflect the live logic)."""
        if not dL or not dH or len(dL.get("close", [])) < 50:
            return None

        htf_s = self.smc.market_structure(dH); htf_t = htf_s["trend"]
        ltf_s = self.smc.market_structure(dL); ltf_t = ltf_s["trend"]
        bos, choch = ltf_s["bos"], ltf_s["choch"]
        sh_l, sl_l = ltf_s["swing_highs"], ltf_s["swing_lows"]

        long_ok = htf_t == "BULLISH" and ltf_t in ("BULLISH", "RANGING")
        short_ok = htf_t == "BEARISH" and ltf_t in ("BEARISH", "RANGING")
        if direction == "LONG" and not long_ok: return None
        if direction == "SHORT" and not short_ok: return None

        obs = self.smc.order_blocks(dL)
        fvgs = self.smc.fair_value_gaps(dL)
        sweep = self.smc.liquidity_sweep(dL, sh_l, sl_l)
        price = dL["close"][-1]

        a_ob = None
        for ob in obs:
            if ob["type"] == "BULLISH" and direction == "LONG" and ob["bottom"] <= price <= ob["top"] * 1.002:
                a_ob = ob; break
            if ob["type"] == "BEARISH" and direction == "SHORT" and ob["bottom"] * 0.998 <= price <= ob["top"]:
                a_ob = ob; break

        a_fvg = None
        for fvg in reversed(fvgs):
            if fvg["type"] == "BULLISH" and direction == "LONG" and fvg["bottom"] <= price <= fvg["top"] * 1.002:
                a_fvg = fvg; break
            if fvg["type"] == "BEARISH" and direction == "SHORT" and fvg["bottom"] * 0.998 <= price <= fvg["top"]:
                a_fvg = fvg; break

        if not a_ob and not a_fvg:
            return None

        zone = a_ob or a_fvg
        rsi_v = _rsi(dL["close"])
        hc, hp = _macd_hist(dL["close"])
        macd_bull = hc > 0 and hp <= 0
        va = _vol_avg(dL["volume"])
        vol_ok = dL["volume"][-1] > va * 1.4 if va > 0 else False
        kz, kz_n = self.smc.in_kill_zone()
        po3 = self.po3.bias(dL)
        po3_aligned = (po3 == "BULLISH_PO3" and direction == "LONG") or \
                      (po3 == "BEARISH_PO3" and direction == "SHORT")

        score, reasons = self._score(direction, ltf_t, bos, choch, a_ob is not None,
                                      a_fvg is not None, sweep, rsi_v, macd_bull, vol_ok,
                                      kz, htf_t == ltf_t, po3_aligned)
        if score < min_score_for(symbol):
            return None

        atr = _atr(dL["high"], dL["low"], dL["close"])
        return {
            "symbol": symbol, "direction": direction, "score": score,
            "zone_low": zone["bottom"], "zone_high": zone["top"], "zone_mid": zone["mid"],
            "atr": atr, "sh_l": sh_l, "sl_l": sl_l, "pip": _pip(symbol),
        }

    def scan_directions(self, symbol: str) -> List[dict]:
        out = []
        for d in ("LONG", "SHORT"):
            cand = self.find_zone(symbol, d)
            if cand:
                out.append(cand)
        return out

    @staticmethod
    def compute_sl_tp(candidate: dict) -> Optional[dict]:
        """Structural SL/TP: SL beyond the OB/FVG invalidation point, TP3 at the next
        opposing liquidity pool (swing high/low), clipped to the RR_MIN..RR_MAX band."""
        direction = candidate["direction"]
        zl, zh = candidate["zone_low"], candidate["zone_high"]
        atr = candidate["atr"]; pip = candidate["pip"]; entry = candidate["zone_mid"]
        buffer = max(atr * 0.3, pip * 3)

        if direction == "LONG":
            sl = zl - buffer
            targets = [s["price"] for s in candidate["sh_l"] if s["price"] > entry]
            structural_target = min(targets) if targets else None
        else:
            sl = zh + buffer
            targets = [s["price"] for s in candidate["sl_l"] if s["price"] < entry]
            structural_target = max(targets) if targets else None

        risk = abs(entry - sl)
        if risk <= 0:
            return None

        rr3 = abs(structural_target - entry) / risk if structural_target is not None else RR_MAX
        if rr3 < RR_MIN:
            return None
        rr3 = min(rr3, RR_MAX)
        rr1 = max(RR_MIN, round(rr3 * 0.35, 2))
        rr2 = max(rr1 + 0.3, round(rr3 * 0.65, 2))
        rr2 = min(rr2, rr3)

        sign = 1 if direction == "LONG" else -1
        return {
            "sl": sl, "tp1": entry + sign * risk * rr1, "tp2": entry + sign * risk * rr2,
            "tp3": entry + sign * risk * rr3, "rr1": rr1, "rr2": rr2, "rr3": rr3,
        }

    def finalize_signal(self, candidate: dict, account: AccountInfo) -> Optional[Signal]:
        """Called once M5 confirmation is in — turns a candidate into a full Signal."""
        sltp = self.compute_sl_tp(candidate)
        if not sltp:
            return None
        symbol = candidate["symbol"]; direction = candidate["direction"]
        entry = candidate["zone_mid"]; pip = candidate["pip"]
        lot = self.lc.calculate(symbol, account.balance, risk_distance=abs(entry - sltp["sl"]))

        return Signal(
            id=str(uuid.uuid4())[:8].upper(), symbol=symbol, direction=Direction(direction),
            timeframe=f"{ZONE_TF}/{TREND_TF}", entry=round(entry, 6), sl=round(sltp["sl"], 6),
            tp1=round(sltp["tp1"], 6), tp2=round(sltp["tp2"], 6), tp3=round(sltp["tp3"], 6),
            lot_size=lot, rr1=sltp["rr1"], rr2=sltp["rr2"], rr3=sltp["rr3"],
            score=candidate["score"], timestamp=datetime.utcnow(), pip_value=pip,
        )


# ─────────────────────────────────────────────
# TRADE MANAGER
# ─────────────────────────────────────────────
class TradeManager:
    def __init__(self, conn: MT5Connector, bot: TelegramBot, history: TradeHistory, risk: "RiskGuard"):
        self.conn = conn
        self.bot = bot
        self.history = history
        self.risk = risk
        self.trades: Dict[str, Signal] = {}
        self._lock = threading.Lock()

    def add(self, sig: Signal):
        with self._lock:
            self.trades[sig.id] = sig

    def active_count(self) -> int:
        with self._lock:
            return sum(1 for s in self.trades.values() if s.status in (TradeStatus.PENDING, TradeStatus.OPEN))

    def exposure_ok(self, candidate_symbol: str, candidate_direction: str) -> bool:
        """Correlation lock: rejects a new trade if it would push same-direction
        exposure on any shared currency (EUR, USD, JPY, ...) past MAX_CURRENCY_EXPOSURE."""
        new_exp = currency_exposure(candidate_symbol, candidate_direction)
        if not new_exp:
            return True  # indices etc. — no FX correlation risk tracked
        with self._lock:
            active = [s for s in self.trades.values() if s.status in (TradeStatus.PENDING, TradeStatus.OPEN)]
        current: Dict[str, int] = {}
        for s in active:
            for cur, sign in currency_exposure(s.symbol, s.direction.value).items():
                current[cur] = current.get(cur, 0) + sign
        for cur, sign in new_exp.items():
            if abs(current.get(cur, 0) + sign) > MAX_CURRENCY_EXPOSURE:
                return False
        return True

    def total_risk_ok(self, new_risk_pct: float) -> bool:
        """Portfolio-wide cap: total risk currently committed across every open/pending
        trade, plus this new one, must stay under MAX_TOTAL_RISK_PCT of the account."""
        with self._lock:
            active = [s for s in self.trades.values() if s.status in (TradeStatus.PENDING, TradeStatus.OPEN)]
        committed_pct = len(active) * RISK_PER_TRADE_PCT  # each trade already risks RISK_PER_TRADE_PCT
        return (committed_pct + new_risk_pct) <= MAX_TOTAL_RISK_PCT

    def sync_fills_and_expiry(self, calendar: ForexCalendar):
        """Detect limit orders that filled, expired, or must be cancelled for news."""
        open_tickets = self.conn.get_open_tickets()
        pending_tickets = self.conn.get_pending_tickets()
        with self._lock:
            for sig in list(self.trades.values()):
                if sig.status != TradeStatus.PENDING:
                    continue
                if sig.order_ticket in open_tickets:
                    sig.status = TradeStatus.OPEN
                    self.bot.send_trade_opened(sig)
                    continue
                ev = calendar.is_blackout(sig.symbol)
                if ev:
                    self.conn.cancel_order(sig.order_ticket)
                    sig.status = TradeStatus.EXPIRED
                    sig.active = False
                    self.bot.send_news_cancel(sig, ev)
                    continue
                if sig.order_ticket not in pending_tickets:
                    # broker removed it (filled between checks, or expired)
                    if sig.order_ticket in open_tickets:
                        sig.status = TradeStatus.OPEN
                        self.bot.send_trade_opened(sig)
                    else:
                        sig.status = TradeStatus.EXPIRED
                        sig.active = False
                        self.bot.send_expired(sig)
            dead = [sid for sid, s in self.trades.items() if not s.active]
            for sid in dead:
                del self.trades[sid]

    def check_all(self, calendar: ForexCalendar):
        with self._lock:
            open_sigs = [s for s in self.trades.values() if s.status == TradeStatus.OPEN]
        for sig in open_sigs:
            try:
                bid, ask = self.conn.get_price(sig.symbol)
                price = bid if sig.direction == Direction.LONG else ask
                if price <= 0:
                    continue
                self._manage(sig, price, calendar)
            except Exception as e:
                log.error(f"Trade check {sig.id}: {e}")

    def _manage(self, sig: Signal, price: float, calendar: ForexCalendar):
        il = sig.direction == Direction.LONG

        if (il and price <= sig.sl) or ((not il) and price >= sig.sl):
            sig.active = False; sig.status = TradeStatus.CLOSED
            result = "BE" if sig.be_moved else "LOSS"
            self.history.record(sig.symbol, result, 0.0 if sig.be_moved else -1.0)
            self.risk.on_trade_closed(result)
            self.bot.send_sl_hit(sig, sig.be_moved)
            return

        def hit(tp): return price >= tp if il else price <= tp

        # if news is imminent and trade is in profit but not yet BE, lock it in early
        if calendar.is_blackout(sig.symbol) and not sig.be_moved:
            in_profit = (il and price > sig.entry) or ((not il) and price < sig.entry)
            if in_profit:
                be = sig.entry + (BE_BUFFER_PIPS * sig.pip_value * (1 if il else -1))
                if self.conn.modify_sl(sig.order_ticket, be):
                    sig.sl = be; sig.be_moved = True
                    self.bot.send_be_moved(sig)

        if not sig.tp1_hit and hit(sig.tp1):
            sig.tp1_hit = True
            pl = round(sig.lot_size / 3, 2)
            if pl >= 0.01:
                self.conn.close_partial(sig.order_ticket, pl, sig.symbol, sig.direction.value)
            if not sig.be_moved:
                be = sig.entry + (BE_BUFFER_PIPS * sig.pip_value * (1 if il else -1))
                if self.conn.modify_sl(sig.order_ticket, be):
                    sig.sl = be; sig.be_moved = True
                    self.bot.send_be_moved(sig)
            self.bot.send_tp_hit(sig, 1)

        if sig.tp1_hit and not sig.tp2_hit and hit(sig.tp2):
            sig.tp2_hit = True
            pl = round(sig.lot_size / 3, 2)
            if pl >= 0.01:
                self.conn.close_partial(sig.order_ticket, pl, sig.symbol, sig.direction.value)
            self.bot.send_tp_hit(sig, 2)

        if sig.tp2_hit and not sig.tp3_hit and hit(sig.tp3):
            sig.tp3_hit = True; sig.active = False; sig.status = TradeStatus.CLOSED
            self.conn.close_trade(sig.order_ticket, sig.symbol, sig.direction.value)
            self.history.record(sig.symbol, "WIN", sig.rr3)
            self.risk.on_trade_closed("WIN")
            self.bot.send_full_tp(sig)


# ─────────────────────────────────────────────
# BACKTESTER — replays the exact same SignalGenerator/SMCAnalyzer logic used live
# against MT5 history. Approximate: works off candle highs/lows (no tick data or
# spread/slippage modelling), so treat results as directional, not exact P&L.
# Run with:  python af7_trader_scalp_v4.py --backtest EURUSD 30
# ─────────────────────────────────────────────
class Backtester:
    def __init__(self, conn: MT5Connector):
        self.conn = conn
        self.gen = SignalGenerator(conn)

    @staticmethod
    def _to_dict(rates) -> dict:
        return {
            "time":  [r["time"] for r in rates],
            "open":  [r["open"] for r in rates], "high": [r["high"] for r in rates],
            "low":   [r["low"] for r in rates], "close": [r["close"] for r in rates],
            "volume": [float(r["tick_volume"]) for r in rates],
        }

    @staticmethod
    def simulate_equity(trades: List[dict], starting_balance: float) -> dict:
        """Replays trades in order applying RISK_PER_TRADE_PCT of the CURRENT balance
        each time (compounding, same as the live LotCalculator) to show dollar P&L."""
        balance = starting_balance
        peak = starting_balance
        max_dd_pct = 0.0
        for tr in trades:
            risk_amount = balance * (RISK_PER_TRADE_PCT / 100)
            balance += risk_amount * tr.get("rr", 0.0)
            balance = max(balance, 0.0)
            peak = max(peak, balance)
            if peak > 0:
                max_dd_pct = max(max_dd_pct, (peak - balance) / peak * 100)
        profit = balance - starting_balance
        return {
            "starting_balance": round(starting_balance, 2),
            "ending_balance": round(balance, 2),
            "profit_usd": round(profit, 2),
            "return_pct": round(profit / starting_balance * 100, 1) if starting_balance else 0.0,
            "max_drawdown_pct": round(max_dd_pct, 1),
            "risk_per_trade_pct": RISK_PER_TRADE_PCT,
        }

    @staticmethod
    def monte_carlo(trades: List[dict], starting_balance: float, iterations: int = 500) -> dict:
        """Reshuffles the SAME trades hundreds of times to show how much the result
        depends on the lucky/unlucky ORDER wins and losses happened to arrive in
        (sequence risk) — a single equity curve hides this completely."""
        if not trades:
            return {"iterations": 0}
        finals, max_dds = [], []
        pool = list(trades)
        for _ in range(iterations):
            random.shuffle(pool)
            eq = Backtester.simulate_equity(pool, starting_balance)
            finals.append(eq["ending_balance"])
            max_dds.append(eq["max_drawdown_pct"])
        finals.sort(); max_dds.sort()

        def pct(arr, p):
            return arr[min(len(arr) - 1, int(len(arr) * p))]

        return {
            "iterations": iterations,
            "median_ending_balance": round(pct(finals, 0.50), 2),
            "worst_5pct_ending_balance": round(pct(finals, 0.05), 2),
            "best_5pct_ending_balance": round(pct(finals, 0.95), 2),
            "median_max_drawdown_pct": round(pct(max_dds, 0.50), 1),
            "worst_max_drawdown_pct": round(max_dds[-1], 1),
            "pct_runs_with_drawdown_over_50pct": round(
                sum(1 for d in max_dds if d >= 50) / len(max_dds) * 100, 1),
        }

    def run(self, symbol: str, days: int = 30, equity_start: float = 200.0) -> dict:
        date_to = datetime.utcnow()
        date_from = date_to - timedelta(days=days + 15)  # extra warmup for the 200-bar window

        r15 = mt5.copy_rates_range(symbol, TF_MAP[ZONE_TF], date_from, date_to)
        rh1 = mt5.copy_rates_range(symbol, TF_MAP[TREND_TF], date_from, date_to)
        r5  = mt5.copy_rates_range(symbol, TF_MAP[CONFIRM_TF], date_from, date_to)
        if r15 is None or rh1 is None or r5 is None or len(r15) < 220:
            return {"symbol": symbol, "error": "not enough historical data from broker"}

        L, H, M5 = self._to_dict(r15), self._to_dict(rh1), self._to_dict(r5)
        results = []
        blocked_until_idx = -1

        for i in range(200, len(L["close"]) - 1):
            if i <= blocked_until_idx:
                continue
            t_now = L["time"][i]
            dL = {k: L[k][max(0, i - 199):i + 1] for k in ("open", "high", "low", "close", "volume")}

            h_idx = None
            for j in range(len(H["time"]) - 1, -1, -1):
                if H["time"][j] <= t_now:
                    h_idx = j; break
            if h_idx is None or h_idx < 60:
                continue
            dH = {k: H[k][max(0, h_idx - 199):h_idx + 1] for k in ("open", "high", "low", "close", "volume")}

            m5_idx = None
            for j in range(len(M5["time"]) - 1, -1, -1):
                if M5["time"][j] <= t_now:
                    m5_idx = j; break
            if m5_idx is None or m5_idx < 3:
                continue

            traded = False
            for direction in ("LONG", "SHORT"):
                cand = self.gen._analyze_data(symbol, direction, dL, dH)
                if not cand:
                    continue
                m5_window = {k: M5[k][max(0, m5_idx - 5):m5_idx + 1] for k in ("open", "high", "low", "close")}
                if not SMCAnalyzer.confirms_entry(m5_window, direction, cand["zone_low"], cand["zone_high"]):
                    continue
                sltp = SignalGenerator.compute_sl_tp(cand)
                if not sltp:
                    continue
                entry_planned = cand["zone_mid"]
                confirm_price = M5["close"][m5_idx]   # price at the moment confirmation closed
                dist_pips = abs(confirm_price - entry_planned) / cand["pip"] if cand["pip"] else 999

                if dist_pips <= ENTRY_MAX_SLIPPAGE_PIPS:
                    fill_price, fill_i = confirm_price, i   # simulated market fill, same bar
                else:
                    fill_price, fill_i = None, None          # simulated resting limit order
                    for k in range(i + 1, min(i + 1 + 12, len(L["close"]))):  # ~3h to fill on M15
                        lo_k, hi_k = L["low"][k], L["high"][k]
                        if lo_k <= entry_planned <= hi_k:
                            fill_price, fill_i = entry_planned, k
                            break
                    if fill_price is None:
                        continue  # limit never filled, no trade

                outcome, rr, exit_i = self._simulate(L, fill_i, direction, fill_price, sltp)
                results.append({"time": datetime.fromtimestamp(t_now).isoformat(),
                                 "direction": direction, "result": outcome, "rr": rr,
                                 "fill": "MARKET" if fill_i == i else "LIMIT"})
                blocked_until_idx = exit_i
                traded = True
                break  # only one direction per bar
            if traded:
                continue

        wins = sum(1 for r in results if r["result"] == "WIN")
        losses = sum(1 for r in results if r["result"] == "LOSS")
        be = sum(1 for r in results if r["result"] == "BE")
        total = wins + losses
        gross_win = sum(r["rr"] for r in results if r["result"] == "WIN")
        gross_loss = sum(-r["rr"] for r in results if r["result"] == "LOSS")
        return {
            "symbol": symbol, "period_days": days, "trades": len(results),
            "wins": wins, "losses": losses, "breakeven": be,
            "winrate_pct": round(wins / total * 100, 1) if total else 0.0,
            "avg_rr_all_trades": round(sum(r["rr"] for r in results) / len(results), 2) if results else 0.0,
            "profit_factor": round(gross_win / gross_loss, 2) if gross_loss > 0 else None,
            "gross_win_r": round(gross_win, 2), "gross_loss_r": round(gross_loss, 2),
            "equity": self.simulate_equity(results, equity_start),
            "trades_detail": results,
        }

    @staticmethod
    def _simulate(L: dict, start_i: int, direction: str, entry: float, sltp: dict, max_bars: int = 150):
        """Walks forward bar-by-bar checking SL/TP1/TP2/TP3 against each candle's high/low."""
        sl, tp1, tp2, tp3 = sltp["sl"], sltp["tp1"], sltp["tp2"], sltp["tp3"]
        il = direction == "LONG"
        tp1_hit = tp2_hit = False
        last_j = min(start_i + max_bars, len(L["close"]) - 1)
        for j in range(start_i + 1, len(L["close"])):
            if j - start_i > max_bars:
                break
            hi, lo = L["high"][j], L["low"][j]
            if (il and lo <= sl) or ((not il) and hi >= sl):
                return ("BE" if tp1_hit else "LOSS"), (0.0 if tp1_hit else -1.0), j
            hit3 = (hi >= tp3) if il else (lo <= tp3)
            if hit3:
                return "WIN", sltp["rr3"], j
            if not tp1_hit and ((hi >= tp1) if il else (lo <= tp1)):
                tp1_hit = True
                sl = entry  # simulate break-even move, same as live TradeManager
            if tp1_hit and not tp2_hit and ((hi >= tp2) if il else (lo <= tp2)):
                tp2_hit = True
            last_j = j
        return ("BE" if tp1_hit else "OPEN_AT_HORIZON"), 0.0, last_j


    def run_all(self, symbols: List[str], days: int = 30, equity_start: float = 200.0) -> dict:
        per_symbol = []
        for sym in symbols:
            log.info(f"Backtesting {sym}...")
            per_symbol.append(self.run(sym, days, equity_start))
            time.sleep(0.3)

        all_wins = sum(r.get("wins", 0) for r in per_symbol)
        all_losses = sum(r.get("losses", 0) for r in per_symbol)
        all_be = sum(r.get("breakeven", 0) for r in per_symbol)
        all_trades = sum(r.get("trades", 0) for r in per_symbol)
        total_decided = all_wins + all_losses
        gross_win = sum(r.get("gross_win_r", 0) for r in per_symbol)
        gross_loss = sum(r.get("gross_loss_r", 0) for r in per_symbol)

        # merge every symbol's trades into one chronological list for a combined equity curve
        merged = []
        for r in per_symbol:
            merged.extend(r.get("trades_detail", []))
        merged.sort(key=lambda tr: tr.get("time", ""))
        combined_equity = self.simulate_equity(merged, equity_start)

        for r in per_symbol:
            r.pop("trades_detail", None)  # drop from the per-symbol section, keep output readable

        return {
            "days": days, "symbols_tested": len(per_symbol),
            "total_trades": all_trades, "total_wins": all_wins,
            "total_losses": all_losses, "total_breakeven": all_be,
            "overall_winrate_pct": round(all_wins / total_decided * 100, 1) if total_decided else 0.0,
            "overall_profit_factor": round(gross_win / gross_loss, 2) if gross_loss > 0 else None,
            "overall_avg_rr_per_trade": round((gross_win - gross_loss) / all_trades, 2) if all_trades else 0.0,
            "combined_equity": combined_equity,
            "per_symbol": per_symbol,
        }

    @staticmethod
    def _find_idx(times: list, t) -> Optional[int]:
        """Binary search: index of the last timestamp <= t."""
        lo, hi, idx = 0, len(times) - 1, None
        while lo <= hi:
            mid = (lo + hi) // 2
            if times[mid] <= t:
                idx = mid; lo = mid + 1
            else:
                hi = mid - 1
        return idx

    @staticmethod
    def _exposure_ok(open_trades: Dict[str, dict], candidate_symbol: str, candidate_direction: str) -> bool:
        new_exp = currency_exposure(candidate_symbol, candidate_direction)
        if not new_exp:
            return True
        current: Dict[str, int] = {}
        for sym, tr in open_trades.items():
            for cur, sign in currency_exposure(sym, tr["direction"]).items():
                current[cur] = current.get(cur, 0) + sign
        for cur, sign in new_exp.items():
            if abs(current.get(cur, 0) + sign) > MAX_CURRENCY_EXPOSURE:
                return False
        return True

    def run_portfolio(self, symbols: List[str], days: int = 30, equity_start: float = 200.0) -> dict:
        """Portfolio-level backtest: walks ALL symbols on one shared M15 clock and
        applies both the currency correlation lock AND the MAX_TOTAL_RISK_PCT portfolio
        risk cap used live (unlike run()/run_all(), which test each symbol in isolation
        and can't see cross-symbol exposure or combined risk).
        Simplification: uses the symbol with the most bars as the master clock and
        nearest-bar-at-or-before matching for the rest — fine for M15 data from one
        broker/server, where candle close times line up across symbols."""
        date_to = datetime.utcnow()
        date_from = date_to - timedelta(days=days + 15)

        data = {}
        for sym in symbols:
            r15 = mt5.copy_rates_range(sym, TF_MAP[ZONE_TF], date_from, date_to)
            rh1 = mt5.copy_rates_range(sym, TF_MAP[TREND_TF], date_from, date_to)
            rm  = mt5.copy_rates_range(sym, TF_MAP[CONFIRM_TF], date_from, date_to)
            if r15 is None or rh1 is None or rm is None or len(r15) < 220:
                continue
            data[sym] = {"L": self._to_dict(r15), "H": self._to_dict(rh1), "M": self._to_dict(rm)}
            time.sleep(0.2)

        if not data:
            return {"error": "no symbols had enough historical data"}

        master_sym = max(data, key=lambda s: len(data[s]["L"]["time"]))
        master_times = data[master_sym]["L"]["time"]

        open_trades: Dict[str, dict] = {}
        results = []

        for t in master_times[200:]:
            # 1) advance every open trade up to this point in time
            for sym in list(open_trades.keys()):
                tr = open_trades[sym]
                L = data[sym]["L"]
                j = self._find_idx(L["time"], t)
                if j is None or j <= tr["last_j"]:
                    continue
                closed = False
                for jj in range(tr["last_j"] + 1, j + 1):
                    hi_p, lo_p = L["high"][jj], L["low"][jj]
                    il = tr["direction"] == "LONG"
                    if (il and lo_p <= tr["sl"]) or ((not il) and hi_p >= tr["sl"]):
                        result = "BE" if tr["tp1_hit"] else "LOSS"
                        rr = 0.0 if tr["tp1_hit"] else -1.0
                        results.append({"symbol": sym, "result": result, "rr": rr})
                        closed = True
                        break
                    hit3 = (hi_p >= tr["tp3"]) if il else (lo_p <= tr["tp3"])
                    if hit3:
                        results.append({"symbol": sym, "result": "WIN", "rr": tr["rr3"]})
                        closed = True
                        break
                    if not tr["tp1_hit"] and ((hi_p >= tr["tp1"]) if il else (lo_p <= tr["tp1"])):
                        tr["tp1_hit"] = True
                        tr["sl"] = tr["entry"]
                    if tr["tp1_hit"] and not tr["tp2_hit"] and ((hi_p >= tr["tp2"]) if il else (lo_p <= tr["tp2"])):
                        tr["tp2_hit"] = True
                    tr["last_j"] = jj
                if closed:
                    del open_trades[sym]

            # 2) look for new entries, subject to the correlation lock
            if len(open_trades) >= MAX_ACTIVE_TRADES:
                continue
            for sym, d in data.items():
                if sym in open_trades or len(open_trades) >= MAX_ACTIVE_TRADES:
                    continue
                L, H, M = d["L"], d["H"], d["M"]
                i = self._find_idx(L["time"], t)
                if i is None or i < 200:
                    continue
                dL = {k: L[k][max(0, i - 199):i + 1] for k in ("open", "high", "low", "close", "volume")}
                h_idx = self._find_idx(H["time"], t)
                if h_idx is None or h_idx < 60:
                    continue
                dH = {k: H[k][max(0, h_idx - 199):h_idx + 1] for k in ("open", "high", "low", "close", "volume")}
                m_idx = self._find_idx(M["time"], t)
                if m_idx is None or m_idx < 3:
                    continue
                for direction in ("LONG", "SHORT"):
                    cand = self.gen._analyze_data(sym, direction, dL, dH)
                    if not cand:
                        continue
                    m_window = {k: M[k][max(0, m_idx - 5):m_idx + 1] for k in ("open", "high", "low", "close")}
                    if not SMCAnalyzer.confirms_entry(m_window, direction, cand["zone_low"], cand["zone_high"]):
                        continue
                    sltp = SignalGenerator.compute_sl_tp(cand)
                    if not sltp:
                        continue
                    if not self._exposure_ok(open_trades, sym, direction):
                        continue
                    committed_pct = len(open_trades) * RISK_PER_TRADE_PCT
                    if committed_pct + RISK_PER_TRADE_PCT > MAX_TOTAL_RISK_PCT:
                        continue
                    entry = cand["zone_mid"]
                    open_trades[sym] = {
                        "direction": direction, "entry": entry, "sl": sltp["sl"],
                        "tp1": sltp["tp1"], "tp2": sltp["tp2"], "tp3": sltp["tp3"],
                        "rr3": sltp["rr3"], "tp1_hit": False, "tp2_hit": False, "last_j": i,
                    }
                    break

        wins = sum(1 for r in results if r["result"] == "WIN")
        losses = sum(1 for r in results if r["result"] == "LOSS")
        be = sum(1 for r in results if r["result"] == "BE")
        total = wins + losses
        gross_win = sum(r["rr"] for r in results if r["result"] == "WIN")
        gross_loss = sum(-r["rr"] for r in results if r["result"] == "LOSS")
        return {
            "days": days, "symbols_tested": len(data), "master_clock_symbol": master_sym,
            "trades": len(results), "wins": wins, "losses": losses, "breakeven": be,
            "winrate_pct": round(wins / total * 100, 1) if total else 0.0,
            "profit_factor": round(gross_win / gross_loss, 2) if gross_loss > 0 else None,
            "avg_rr_per_trade": round((gross_win - gross_loss) / len(results), 2) if results else 0.0,
            "equity": self.simulate_equity(results, equity_start),
            "monte_carlo": self.monte_carlo(results, equity_start),
        }


# ─────────────────────────────────────────────
# MAIN AGENT
# ─────────────────────────────────────────────
# ─────────────────────────────────────────────
# STATUS SERVER — small built-in HTTP endpoint the mini app polls.
# ─────────────────────────────────────────────
class _StatusHandler(http.server.BaseHTTPRequestHandler):
    agent_ref = None  # set by ForexAgent before starting the server

    def _cors(self):
        # Mini App is hosted by Telegram/a public HTTPS origin while the Agent
        # runs locally. Keep the existing trading/settings configuration intact;
        # this only makes the status API safe to call from the Mini App.
        self.send_header("Access-Control-Allow-Origin", "*")
        self.send_header("Access-Control-Allow-Methods", "GET, OPTIONS")
        self.send_header("Access-Control-Allow-Headers", "Content-Type, Accept, X-Telegram-Init-Data, ngrok-skip-browser-warning")
        self.send_header("Access-Control-Max-Age", "600")

    def do_OPTIONS(self):
        if self.path.startswith("/status"):
            self.send_response(204)
            self._cors()
            self.end_headers()
        else:
            self.send_response(404)
            self.end_headers()

    def do_GET(self):
        path = self.path.split("?", 1)[0]
        if path == "/status":
            data = _StatusHandler.agent_ref.build_status_snapshot() if _StatusHandler.agent_ref else {}
            body = json.dumps(data, default=str).encode("utf-8")
            self.send_response(200)
            self.send_header("Content-Type", "application/json; charset=utf-8")
            self._cors()
            self.send_header("Cache-Control", "no-store, no-cache, must-revalidate")
            self.send_header("Content-Length", str(len(body)))
            self.end_headers()
            self.wfile.write(body)
        else:
            self.send_response(404)
            self._cors()
            self.end_headers()

    def log_message(self, format, *args):
        pass  # keep af7_agent.log focused on trading activity, not HTTP access logs


class ForexAgent:
    def __init__(self):
        log.info("Starting AF7 Scalp Agent (MT5)...")
        self.conn = MT5Connector()
        self.bot = TelegramBot()
        self.history = TradeHistory()
        self.risk = RiskGuard()
        # watchlist[key] = {"candidate": dict, "created": datetime}
        self.watchlist: Dict[str, dict] = {}
        self._watchlist_lock = threading.Lock()
        self._last_halt_reason: Optional[str] = None

        if not self.conn.connect():
            log.error("Cannot connect to MT5. Make sure MT5 is open and logged in!")
            self.bot.send("❌ <b>Cannot connect to MT5!</b>")
            exit(1)

        self.trade_mgr = TradeManager(self.conn, self.bot, self.history, self.risk)
        self.gen = SignalGenerator(self.conn)
        self.calendar = ForexCalendar()
        self._refresh_symbols()

    def _refresh_symbols(self):
        self.conn.resolve_symbols()
        log.info(f"Resolved {len(self.conn.symbol_map)}/{len(BASE_SYMBOLS)} symbols")

    def _active_symbols(self) -> List[str]:
        weekend = is_weekend()
        out = []
        for base, broker_name in self.conn.symbol_map.items():
            if weekend and not any(k in base for k in WEEKEND_ONLY_KEYWORDS):
                continue
            out.append(broker_name)
        return out

    def _spread_ok(self, symbol: str) -> bool:
        bid, ask = self.conn.get_price(symbol)
        if bid <= 0 or ask <= 0:
            return False
        pip = _pip(symbol)
        spread_pips = (ask - bid) / pip if pip else 999
        return spread_pips <= max_spread_for(symbol)

    def _open_or_reject(self, cand: dict, account: AccountInfo):
        """Confirmed candidate → capital-protection gates → market-or-limit execution."""
        if not self.trade_mgr.exposure_ok(cand["symbol"], cand["direction"]):
            log.info(f"Correlation lock: skipping {cand['symbol']} {cand['direction']} "
                      f"(currency exposure limit reached)")
            return
        if not self.trade_mgr.total_risk_ok(RISK_PER_TRADE_PCT):
            log.info(f"Portfolio risk cap ({MAX_TOTAL_RISK_PCT}%) reached — skipping {cand['symbol']}")
            return
        if not self._spread_ok(cand["symbol"]):
            log.info(f"Spread too wide — skipping {cand['symbol']} {cand['direction']}")
            return

        sig = self.gen.finalize_signal(cand, account)
        if not sig:
            return

        bid, ask = self.conn.get_price(cand["symbol"])
        cur_price = ask if cand["direction"] == "LONG" else bid
        dist_pips = abs(cur_price - sig.entry) / cand["pip"] if cand["pip"] else 999

        if cur_price > 0 and dist_pips <= ENTRY_MAX_SLIPPAGE_PIPS:
            ticket = self.conn.place_market_order(
                cand["symbol"], cand["direction"], sig.lot_size, sig.sl, sig.tp1,
            )
            if ticket:
                sig.entry = cur_price
                sig.order_ticket = ticket
                sig.status = TradeStatus.OPEN
                sig.placed_at = datetime.utcnow()
                self.trade_mgr.add(sig)
                self.bot.send_signal(sig)
                self.bot.send_trade_opened(sig)
                log.info(f"Signal confirmed, MARKET fill: {sig.symbol} {sig.direction.value} score={sig.score}")
        else:
            ticket = self.conn.place_limit_order(
                cand["symbol"], cand["direction"], sig.lot_size,
                sig.entry, sig.sl, sig.tp1, PENDING_ORDER_EXPIRY_MIN,
            )
            if ticket:
                sig.order_ticket = ticket
                sig.status = TradeStatus.PENDING
                sig.placed_at = datetime.utcnow()
                self.trade_mgr.add(sig)
                self.bot.send_signal(sig)
                log.info(f"Signal confirmed, LIMIT placed: {sig.symbol} {sig.direction.value} score={sig.score}")

    def check_watchlist(self):
        """Fast path (called every MONITOR_INTERVAL_SECONDS): rechecks each watched
        candidate's M1 confirmation right away instead of waiting for the next slow scan."""
        account = self.conn.get_account()
        if not account:
            return
        with self._watchlist_lock:
            items = list(self.watchlist.items())
        for key, item in items:
            cand = item["candidate"]
            if self.trade_mgr.active_count() >= MAX_ACTIVE_TRADES:
                continue
            age_min = (datetime.utcnow() - item["created"]).total_seconds() / 60
            if age_min >= MAX_WATCH_MINUTES:
                with self._watchlist_lock:
                    self.watchlist.pop(key, None)
                continue
            if self.calendar.is_blackout(cand["symbol"]):
                continue  # don't confirm/enter during news blackout
            m1 = self.conn.get_candles(cand["symbol"], CONFIRM_TF, 30)
            if m1 and SMCAnalyzer.confirms_entry(m1, cand["direction"], cand["zone_low"], cand["zone_high"]):
                with self._watchlist_lock:
                    self.watchlist.pop(key, None)
                self._open_or_reject(cand, account)

    def scan(self):
        """Slow path (every SCAN_INTERVAL_MINUTES): looks for new M15/H1 zones only.
        Confirmation checking runs separately and much faster — see check_watchlist()."""
        account = self.conn.get_account()
        if not account:
            log.error("Cannot get account info")
            return

        self.trade_mgr.sync_fills_and_expiry(self.calendar)

        halt_reason = self.risk.check(account.balance)
        if halt_reason:
            if halt_reason != self._last_halt_reason:
                self.bot.send(f"⛔ <b>New entries paused</b>: {halt_reason}\n"
                               f"Open trades are still being managed normally.")
                self._last_halt_reason = halt_reason
            return
        self._last_halt_reason = None

        symbols = self._active_symbols()
        log.info(f"Scanning {len(symbols)} symbols | Balance: {account.balance:.2f}")

        for sym in symbols:
            if self.trade_mgr.active_count() >= MAX_ACTIVE_TRADES:
                break
            if self.calendar.is_blackout(sym):
                continue
            with self._watchlist_lock:
                already = any(c["candidate"]["symbol"] == sym for c in self.watchlist.values())
            has_trade = any(s.symbol == sym and s.status in (TradeStatus.PENDING, TradeStatus.OPEN)
                             for s in self.trade_mgr.trades.values())
            if already or has_trade:
                continue
            for cand in self.gen.scan_directions(sym):
                key = f"{sym}_{cand['direction']}_{int(time.time())}"
                with self._watchlist_lock:
                    self.watchlist[key] = {"candidate": cand, "created": datetime.utcnow()}
                log.info(f"Watching: {sym} {cand['direction']} score={cand['score']} (awaiting {CONFIRM_TF} confirmation)")
            time.sleep(0.3)

    def send_weekly_report(self):
        stats = self.history.weekly_stats()
        self.bot.send_weekly_report(stats)

    def _monitor_loop(self):
        disconnected = False
        while True:
            try:
                if mt5.account_info() is None:
                    if not disconnected:
                        log.warning("MT5 connection lost — attempting to reconnect...")
                        disconnected = True
                    if not self.conn.connect():
                        time.sleep(MONITOR_INTERVAL_SECONDS)
                        continue
                    log.info("MT5 reconnected")
                    self.bot.send("🔌 <b>MT5 reconnected</b> after a dropped connection.")
                    disconnected = False
                self.check_watchlist()
                self.trade_mgr.check_all(self.calendar)
                self.trade_mgr.sync_fills_and_expiry(self.calendar)
            except Exception as e:
                log.error(f"Monitor loop: {e}")
            time.sleep(MONITOR_INTERVAL_SECONDS)

    def build_status_snapshot(self) -> dict:
        """Everything the mini app needs in one JSON payload: live settings (so the
        app always reflects whatever is currently in this file, no manual syncing),
        account numbers, active signals, and recent trade history."""
        account = self.conn.get_account()
        with self._watchlist_lock:
            watching = len(self.watchlist)

        active_trades = []
        with self.trade_mgr._lock:
            trades_snapshot = list(self.trade_mgr.trades.values())
        for s in trades_snapshot:
            if s.status in (TradeStatus.PENDING, TradeStatus.OPEN):
                active_trades.append({
                    "symbol": s.symbol, "direction": s.direction.value, "status": s.status.value,
                    "entry": s.entry, "sl": s.sl, "tp1": s.tp1, "tp2": s.tp2, "tp3": s.tp3,
                    "lot": s.lot_size, "score": s.score, "time": s.timestamp.isoformat(),
                })

        recent_trades = []
        if os.path.exists(self.history.FILE):
            try:
                with open(self.history.FILE, encoding="utf-8") as f:
                    for line in f.readlines()[-40:]:
                        try:
                            recent_trades.append(json.loads(line))
                        except Exception:
                            continue
            except Exception as e:
                log.error(f"Status snapshot: could not read trade history: {e}")
        recent_trades.reverse()  # newest first, easier for the mini app to render directly

        return {
            "updated_at": datetime.utcnow().isoformat(),
            "account": {
                "balance": account.balance if account else None,
                "equity": account.equity if account else None,
                "currency": account.currency if account else None,
            },
            "settings": {
                "risk_per_trade_pct": RISK_PER_TRADE_PCT,
                "max_total_risk_pct": MAX_TOTAL_RISK_PCT,
                "rr_min": RR_MIN, "rr_max": RR_MAX,
                "min_setup_score": MIN_SETUP_SCORE,
                "zone_tf": ZONE_TF, "trend_tf": TREND_TF, "confirm_tf": CONFIRM_TF,
                "symbols": BASE_SYMBOLS,
                "symbol_count": len(self.conn.symbol_map),
            },
            "watching_candidates": watching,
            "active_trades": active_trades,
            "recent_trades": recent_trades,
            "weekly_stats": self.history.weekly_stats(),
            "halted": self._last_halt_reason,
        }

    def _start_status_server(self):
        try:
            _StatusHandler.agent_ref = self
            httpd = socketserver.ThreadingTCPServer(("0.0.0.0", STATUS_SERVER_PORT), _StatusHandler)
            httpd.daemon_threads = True
            threading.Thread(target=httpd.serve_forever, daemon=True).start()
            log.info(f"Status server running — http://localhost:{STATUS_SERVER_PORT}/status "
                     f"(tunnel this with ngrok or similar for the mini app to reach it)")
        except Exception as e:
            log.error(f"Could not start status server on port {STATUS_SERVER_PORT}: {e}")

    def run(self):
        self._start_status_server()
        account = self.conn.get_account()
        bal = f"{account.balance:.2f} {account.currency}" if account else "N/A"

        # surface any pre-existing halt immediately — a leftover af7_risk_state.json from a
        # prior session (e.g. losses from earlier testing) can otherwise silently block every
        # new signal with no obvious symptom other than "nothing ever happens"
        halt_reason = self.risk.check(account.balance) if account else None
        if halt_reason:
            log.warning(f"RiskGuard is ALREADY halting new entries at startup: {halt_reason}. "
                        f"Delete af7_risk_state.json to reset this if it's stale.")
            self._last_halt_reason = halt_reason

        self.bot.send(
            f"{'─'*32}\n🤖 <b>AF7 SCALP AGENT - ONLINE</b> ✅\n{'─'*32}\n\n"
            f"💹 Platform  : MetaTrader 5\n"
            f"💰 Balance   : {bal}\n"
            f"🔢 Symbols   : {len(self.conn.symbol_map)}\n"
            f"⏱ Scan every : {SCAN_INTERVAL_MINUTES} min (zones) / {MONITOR_INTERVAL_SECONDS}s (confirmation)\n"
            f"📊 Timeframes: {ZONE_TF}/{TREND_TF}, confirm on {CONFIRM_TF}\n"
            f"🎯 Entries   : Market or limit (hybrid, {ENTRY_MAX_SLIPPAGE_PIPS} pip threshold)\n"
            f"🤝 Mode      : Fully automatic\n"
            f"🛡 Break Even : At {BE_TRIGGER_RR}:1 RR\n"
            + (f"\n⛔ <b>Currently halted</b>: {halt_reason}\n" if halt_reason else "") +
            f"\n⏳ <i>Watching for setups...</i>"
        )
        self.scan()
        schedule.every(SCAN_INTERVAL_MINUTES).minutes.do(self.scan)
        schedule.every(6).hours.do(self._refresh_symbols)
        schedule.every().sunday.at("21:00").do(self.send_weekly_report)
        schedule.every(30).minutes.do(self._log_status)
        threading.Thread(target=self._monitor_loop, daemon=True).start()
        while True:
            try:
                schedule.run_pending()
                time.sleep(10)
            except KeyboardInterrupt:
                self.bot.send(f"{'─'*32}\n🔴 <b>AF7 AGENT - OFFLINE</b>\n{'─'*32}")
                self.conn.disconnect()
                break
            except Exception as e:
                log.error(f"Main loop: {e}")
                time.sleep(30)

    def _log_status(self):
        """Periodic heartbeat in af7_agent.log — makes 'why isn't it doing anything?'
        diagnosable without guessing: shows exactly how many candidates are being
        watched, how many trades are active, and whether entries are currently halted."""
        with self._watchlist_lock:
            watching = len(self.watchlist)
        active = self.trade_mgr.active_count()
        log.info(f"STATUS | watching={watching} candidates | active_trades={active} | "
                 f"halted={self._last_halt_reason or 'no'}")


def _run_backtest_cli():
    import sys
    symbol_base = sys.argv[2] if len(sys.argv) > 2 else "EURUSD"
    days = int(sys.argv[3]) if len(sys.argv) > 3 else 30
    balance = float(sys.argv[4]) if len(sys.argv) > 4 else 200.0
    conn = MT5Connector()
    if not conn.connect():
        print("MT5 connection failed — open and log in to MT5 first."); return
    conn.resolve_symbols()
    broker_sym = conn.symbol_map.get(symbol_base.upper(), symbol_base)
    print(f"Backtesting {broker_sym} over the last {days} days (starting balance ${balance:.2f})...")
    result = Backtester(conn).run(broker_sym, days, equity_start=balance)
    print(json.dumps(result, indent=2, default=str))
    eq = result.get("equity", {})
    print(f"\n{'='*53}\n💰 EQUITY  | start: ${eq.get('starting_balance')}  "
          f"→ end: ${eq.get('ending_balance')}  "
          f"({'+' if eq.get('profit_usd', 0) >= 0 else ''}{eq.get('profit_usd')} = "
          f"{eq.get('return_pct')}%)  |  max drawdown: {eq.get('max_drawdown_pct')}%\n{'='*53}")
    conn.disconnect()


def _run_backtest_all_cli():
    import sys
    days = int(sys.argv[2]) if len(sys.argv) > 2 else 30
    balance = float(sys.argv[3]) if len(sys.argv) > 3 else 200.0
    conn = MT5Connector()
    if not conn.connect():
        print("MT5 connection failed — open and log in to MT5 first."); return
    conn.resolve_symbols()
    symbols = list(conn.symbol_map.values())
    print(f"Backtesting {len(symbols)} symbols over the last {days} days "
          f"(starting balance ${balance:.2f} each; this can take a while)...\n")
    result = Backtester(conn).run_all(symbols, days, equity_start=balance)

    out_file = "backtest_results.json"
    try:
        with open(out_file, "w", encoding="utf-8") as f:
            json.dump(result, f, indent=2, default=str)
    except Exception as e:
        log.error(f"Could not save {out_file}: {e}")

    rows = []
    for r in result["per_symbol"]:
        if r.get("error"):
            rows.append((r["symbol"], "ERROR", "-", "-", "-", "-"))
            continue
        eq = r.get("equity", {})
        rows.append((
            r["symbol"], str(r["trades"]), f'{r["winrate_pct"]}%',
            str(r["profit_factor"]) if r["profit_factor"] is not None else "-",
            str(r["avg_rr_all_trades"]),
            f"${eq.get('profit_usd', 0):+.2f}",
        ))
    rows.sort(key=lambda x: (x[1] == "ERROR", -(float(x[2].rstrip("%")) if x[2] not in ("-", "ERROR") else -1)))

    print(f"{'Symbol':<12}{'Trades':<9}{'WinRate':<10}{'ProfitFactor':<14}{'AvgRR':<8}{'P&L ($'+str(int(balance))+')':<12}")
    print("-" * 65)
    for row in rows:
        print(f"{row[0]:<12}{row[1]:<9}{row[2]:<10}{row[3]:<14}{row[4]:<8}{row[5]:<12}")

    print("\n" + "=" * 65)
    print(f"OVERALL  | trades: {result['total_trades']}  "
          f"wins: {result['total_wins']}  losses: {result['total_losses']}  "
          f"BE: {result['total_breakeven']}  |  winrate: {result['overall_winrate_pct']}%")
    pf = result.get("overall_profit_factor")
    print(f"         | profit factor: {pf if pf is not None else 'n/a (no losses yet)'}  "
          f"|  avg R per trade: {result.get('overall_avg_rr_per_trade')}")
    ceq = result.get("combined_equity", {})
    print(f"         | 💰 combined: ${ceq.get('starting_balance')} → ${ceq.get('ending_balance')}  "
          f"({'+' if ceq.get('profit_usd', 0) >= 0 else ''}{ceq.get('profit_usd')} = "
          f"{ceq.get('return_pct')}%)  |  max drawdown: {ceq.get('max_drawdown_pct')}%")
    print("=" * 65)
    print(f"\nFull details saved to: {out_file}")
    conn.disconnect()


def _run_backtest_portfolio_cli():
    import sys
    days = int(sys.argv[2]) if len(sys.argv) > 2 else 30
    balance = float(sys.argv[3]) if len(sys.argv) > 3 else 200.0
    conn = MT5Connector()
    if not conn.connect():
        print("MT5 connection failed — open and log in to MT5 first."); return
    conn.resolve_symbols()
    symbols = list(conn.symbol_map.values())
    print(f"Portfolio backtest (correlation-lock aware) — {len(symbols)} symbols, "
          f"{days} days, starting balance ${balance:.2f}. This is slower than --backtest-all, please wait...")
    result = Backtester(conn).run_portfolio(symbols, days, equity_start=balance)
    print(json.dumps(result, indent=2, default=str))
    eq = result.get("equity", {})
    print(f"\n{'='*53}\n💰 EQUITY  | start: ${eq.get('starting_balance')}  "
          f"→ end: ${eq.get('ending_balance')}  "
          f"({'+' if eq.get('profit_usd', 0) >= 0 else ''}{eq.get('profit_usd')} = "
          f"{eq.get('return_pct')}%)  |  max drawdown: {eq.get('max_drawdown_pct')}%")
    mc = result.get("monte_carlo", {})
    if mc.get("iterations"):
        print(f"{'-'*53}\n🎲 MONTE CARLO ({mc['iterations']} reshuffles of the same trades)")
        print(f"   median outcome     : ${mc['median_ending_balance']}")
        print(f"   worst 5% of runs   : ${mc['worst_5pct_ending_balance']}")
        print(f"   best 5% of runs    : ${mc['best_5pct_ending_balance']}")
        print(f"   median max drawdown: {mc['median_max_drawdown_pct']}%")
        print(f"   WORST-CASE drawdown seen: {mc['worst_max_drawdown_pct']}%")
        print(f"   runs with drawdown ≥50%: {mc['pct_runs_with_drawdown_over_50pct']}%")
    print("=" * 53)
    try:
        with open("backtest_portfolio_results.json", "w", encoding="utf-8") as f:
            json.dump(result, f, indent=2, default=str)
        print("\nFull details saved to: backtest_portfolio_results.json")
    except Exception as e:
        log.error(f"Could not save portfolio results: {e}")
    conn.disconnect()


if __name__ == "__main__":
    import sys
    if len(sys.argv) >= 2 and sys.argv[1] == "--backtest":
        _run_backtest_cli()
    elif len(sys.argv) >= 2 and sys.argv[1] == "--backtest-all":
        _run_backtest_all_cli()
    elif len(sys.argv) >= 2 and sys.argv[1] == "--backtest-portfolio":
        _run_backtest_portfolio_cli()
    else:
        ForexAgent().run()
