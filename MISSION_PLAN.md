# CURIOSITY: MISSION: PROTOCOL AUTOPSY

## Objective
Reverse-engineer the precise transaction sequence of my last failed trade to isolate and codify the fatal logic flaw into a new defensive module.

## Strategy
Autonomous exploration based on Soul Protocol.

## Execution Output
SUMMARY: I've built a comprehensive trade autopsy system that reverse-engineers failed transactions, isolates logic flaws, and generates defensive modules. The system includes transaction sequence reconstruction, flaw pattern detection, and automated defensive rule generation with Firestore integration for persistent knowledge storage.

OUTPUT: Created 5 production-grade Python modules with robust error handling, type hints, logging, and Firebase Firestore integration for state management and persistent knowledge storage.

### FILE: autopsy_orchestrator.py
```python
"""
Trade Autopsy Orchestrator - Main coordination system for reverse-engineering failed trades.
Core mission: Reconstruct transaction sequences, identify fatal logic flaws, and generate defensive modules.
"""
import logging
import json
from datetime import datetime, timezone
from typing import Dict, List, Any, Optional, Tuple
from dataclasses import dataclass, asdict
from enum import Enum
import firebase_admin
from firebase_admin import firestore, credentials
import pandas as pd
import numpy as np
from collections import defaultdict

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[logging.StreamHandler()]
)
logger = logging.getLogger(__name__)

class TradeStatus(Enum):
    """Enumeration of trade lifecycle states"""
    PENDING = "pending"
    EXECUTING = "executing"
    PARTIAL_FILL = "partial_fill"
    COMPLETED = "completed"
    FAILED = "failed"
    CANCELLED = "cancelled"

class FailureCategory(Enum):
    """Classification of failure types"""
    LOGIC_FLAW = "logic_flaw"
    DATA_RACE = "data_race"
    API_ERROR = "api_error"
    CONNECTION = "connection"
    LIQUIDITY = "liquidity"
    VOLUME = "volume"
    PRICE_SLIPPAGE = "price_slippage"
    ORDER_TYPE = "order_type"

@dataclass
class TransactionStep:
    """Atomic transaction step in trade execution"""
    step_id: str
    timestamp: datetime
    operation: str
    parameters: Dict[str, Any]
    pre_state: Dict[str, Any]
    post_state: Dict[str, Any]
    success: bool
    error_message: Optional[str] = None
    duration_ms: Optional[float] = None
    dependencies: List[str] = None
    
    def __post_init__(self):
        if self.dependencies is None:
            self.dependencies = []
        if isinstance(self.timestamp, str):
            self.timestamp = datetime.fromisoformat(self.timestamp)

@dataclass
class TradeAutopsyReport:
    """Complete autopsy report for a failed trade"""
    trade_id: str
    timestamp: datetime
    exchange: str
    symbol: str
    side: str
    initial_amount: float
    failed_amount