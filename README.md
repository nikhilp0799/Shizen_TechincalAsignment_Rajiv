# Shizen_TechincalAsignment_Rajiv

Universal Portfolio Upload Function - Technical Assignment
🎯 Assignment Overview
Welcome! You're applying for a backend engineering position at our fintech company. This assignment tests your ability to extend existing systems while maintaining backward compatibility—a critical skill for our team.

Time Allocation: 4-6 hours
Evaluation Focus: Code quality, extensibility, error handling, backward compatibility

📋 Table of Contents
Background & Context
Current System
Your Mission
Task Breakdown
Evaluation Criteria
Test Data
Submission Guidelines
FAQ
🏢 Background & Context
The Business Problem
Our company manages investment portfolios for institutional clients. Currently, our system only handles equity portfolios at the individual security level. We need to expand to support:

Multiple Asset Classes:
Equities (existing)
Municipal Bonds (new)
Sovereign Bonds (new)
Portfolio Structures:
Individual securities (existing)
Funds (collection of individual securities)
Fund of Funds (collection of funds)
Mixed portfolios (any combination)
Portfolio Levels:
Individual holdings
Portfolio-level aggregations
Multi-tier fund structures
Key Definitions
Portfolio Hierarchy:

Individual Securities Portfolio
├── AAPL (equity)
├── MSFT (equity)
└── 037833100 (municipal bond)

Fund Portfolio
├── VFIAX (fund)
│   ├── AAPL
│   ├── MSFT
│   └── GOOGL
└── VBMFX (fund)
    ├── 037833100 (bond)
    └── 313369AJ3 (bond)

Fund of Funds
└── STRATEGIC_2025 (fund of funds)
    ├── VFIAX (fund)
    │   ├── AAPL
    │   └── MSFT
    └── VGTSX (fund)
        ├── TSM
        └── EFA
🔧 Current System
Existing Code Structure
The repository contains a working equity-only portfolio upload system. Your job is to extend it WITHOUT breaking existing functionality.

models.py - Current Database Model
python
"""
Current database model - HANDLES EQUITIES ONLY
⚠️ DO NOT DELETE OR BREAK THIS MODEL
✅ YOU MUST EXTEND IT
"""

from sqlalchemy import Column, String, Float, DateTime
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime

Base = declarative_base()

class Company(Base):
    """
    Current model for equity securities
    Used by existing production portfolios - DO NOT BREAK!
    """
    __tablename__ = 'companies'
    
    # Primary identification
    id = Column(String, primary_key=True)
    ticker = Column(String, unique=True, nullable=False)
    name = Column(String)
    market_value = Column(Float)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    # Climate/ESG metrics (existing feature)
    carbon_intensity = Column(Float)  # tons CO2 per $M revenue
    climate_score = Column(Float)     # 0-10 scale
portfolio_service.py - Current Upload Function
python
"""
Current portfolio upload implementation
⚠️ THIS FUNCTION IS IN PRODUCTION - DO NOT DELETE
✅ YOU MUST EXTEND IT TO HANDLE ALL ASSET TYPES
"""

import pandas as pd
from typing import Dict, List
from models import Company

def upload_portfolio(file_path: str, portfolio_id: str) -> Dict:
    """
    Current implementation - handles ONLY equity portfolios
    
    This function is used by 50+ existing clients.
    Their CSV files must continue to work unchanged.
    
    Args:
        file_path: Path to CSV file with columns: Ticker, MarketValue
        portfolio_id: Unique portfolio identifier
    
    Returns:
        Dict with status and created companies
        
    Current Expected CSV Format:
        Ticker,MarketValue
        AAPL,150000000
        MSFT,120000000
    """
    df = pd.read_csv(file_path)
    
    # Validate required columns for equity
    if 'Ticker' not in df.columns or 'MarketValue' not in df.columns:
        raise ValueError("Missing required columns: Ticker, MarketValue")
    
    created_companies = []
    
    for _, row in df.iterrows():
        ticker = row['Ticker'].upper().strip()
        market_value = float(row['MarketValue'])
        
        # Fetch equity data from external API
        company_data = fetch_equity_data(ticker)
        
        # Create company record
        company = Company(
            id=f"equity_{ticker}",
            ticker=ticker,
            name=company_data.get('name'),
            market_value=market_value,
            carbon_intensity=company_data.get('carbon_intensity'),
            climate_score=company_data.get('climate_score')
        )
        
        created_companies.append(company)
    
    return {
        'status': 'success',
        'portfolio_id': portfolio_id,
        'companies': created_companies,
        'count': len(created_companies)
    }


def fetch_equity_data(ticker: str) -> Dict:
    """
    Simulates API call to equity data provider
    In production, this calls a real financial data API
    """
    # Mock implementation for testing
    return {
        'name': f"Company {ticker}",
        'sector': 'Technology',
        'carbon_intensity': 150.5,
        'climate_score': 7.2
    }
🎯 Your Mission
Transform the equity-only system into a universal portfolio upload function that handles:

✅ Existing equity CSV files (backward compatible)
✅ Bond-only portfolios (new)
✅ Mixed equity + bond portfolios (new)
✅ Fund portfolios (new)
✅ Fund of Funds portfolios (new)
✅ Any combination of the above (new)
Critical Requirements
Backward Compatibility: Existing equity CSV files MUST continue to work unchanged
Single Function: ONE universal upload function, not multiple functions
Single Table: ONE extended database table, not separate tables per asset type
Extensibility: Easy to add new asset types in the future
Portfolio Hierarchy: Support individual securities, funds, and fund of funds
📝 Task Breakdown
Task 1: Database Schema Extension (30 minutes)
Objective: Extend the Company model to support all asset types WITHOUT breaking existing equity functionality.

Requirements:
Add fields for bond securities:
cusip (VARCHAR 9) - for municipal bonds
isin (VARCHAR 12) - for sovereign bonds
maturity_date (DATE)
credit_rating (VARCHAR 10) - e.g., "AAA", "AA+"
coupon_rate (DOUBLE) - annual interest rate
yield_to_worst (DOUBLE)
duration (DOUBLE) - interest rate sensitivity
Add fields to distinguish security types:
security_type (VARCHAR 50) - "equity", "municipal_bond", "sovereign_bond", "fund"
identifier_type (VARCHAR 20) - "ticker", "cusip", "isin"
Add fields for fund structures:
is_fund (BOOLEAN) - true if this is a fund
parent_fund_id (VARCHAR) - for fund of funds structures
fund_composition (JSON/TEXT) - stores underlying holdings
Maintain backward compatibility:
All new columns must be optional/nullable
Existing equity records must remain valid
Default values must not break existing queries
Deliverables:
Updated models.py with extended Company model
SQL migration script (migration_001_add_bond_support.sql)
Documentation explaining your design decisions
Example Migration Script Structure:
sql
-- migration_001_add_bond_support.sql
-- Add bond and fund support to companies table

-- Add security type fields
ALTER TABLE companies ADD COLUMN security_type VARCHAR(50) DEFAULT 'equity';
ALTER TABLE companies ADD COLUMN identifier_type VARCHAR(20) DEFAULT 'ticker';

-- Add bond-specific fields (all nullable)
ALTER TABLE companies ADD COLUMN cusip VARCHAR(9);
ALTER TABLE companies ADD COLUMN isin VARCHAR(12);
ALTER TABLE companies ADD COLUMN maturity_date DATE;
-- ... continue for all fields

-- Add indexes for performance
CREATE INDEX idx_company_security_type ON companies(security_type);
CREATE INDEX idx_company_cusip ON companies(cusip);
CREATE INDEX idx_company_isin ON companies(isin);

-- Update existing records to have explicit type
UPDATE companies SET security_type='equity', identifier_type='ticker' 
WHERE security_type IS NULL;
Task 2: Security Type Detection (15 minutes)
Objective: Implement automatic identifier detection logic.

The system needs to automatically detect whether an identifier is a ticker, CUSIP, or ISIN based on its format.

Detection Rules:
Identifier	Length	Format	Type	Example
Ticker	1-5 chars	Letters only	Equity	AAPL, MSFT
CUSIP	9 chars	Alphanumeric	Municipal Bond	037833100
ISIN	12 chars	2 letters + 10 alphanumeric	Sovereign Bond	US0378331005
Implementation:
python
def detect_security_type(identifier: str) -> tuple:
    """
    Automatically detect security type from identifier format
    
    Args:
        identifier: String that could be Ticker, CUSIP, or ISIN
    
    Returns:
        Tuple of (security_type, identifier_type)
        
    Examples:
        "AAPL" -> ("equity", "ticker")
        "037833100" -> ("municipal_bond", "cusip")
        "US0378331005" -> ("sovereign_bond", "isin")
    
    Edge Cases to Handle:
        - Empty strings
        - Whitespace
        - Mixed case
        - Invalid formats
        - Null/None values
    """
    # YOUR IMPLEMENTATION HERE
    pass


# Test cases you should pass:
def test_detection():
    assert detect_security_type("AAPL") == ("equity", "ticker")
    assert detect_security_type("aapl") == ("equity", "ticker")  # case insensitive
    assert detect_security_type("037833100") == ("municipal_bond", "cusip")
    assert detect_security_type("US0378331005") == ("sovereign_bond", "isin")
    assert detect_security_type("  MSFT  ") == ("equity", "ticker")  # whitespace
    # Add more edge cases...
Task 3: Universal Upload Function (2-3 hours)
Objective: Transform upload_portfolio into a universal function that handles ALL scenarios.

Supported CSV Formats:
The function must handle multiple CSV format variations:

Format 1: Equity-only (EXISTING - MUST REMAIN WORKING)
csv
Ticker,MarketValue
AAPL,150000000
MSFT,120000000
GOOGL,180000000
Format 2: Bond-only (NEW)
csv
CUSIP,MarketValue
037833100,100000000
313369AJ3,80000000
Format 3: Mixed with generic identifier (NEW)
csv
Identifier,MarketValue
AAPL,150000000
037833100,100000000
MSFT,120000000
US91282CHX4,90000000
Format 4: Fund Portfolio (NEW)
csv
FundIdentifier,MarketValue,Holdings
VFIAX,500000000,"AAPL,MSFT,GOOGL,JPM,JNJ"
Format 5: Fund of Funds (NEW)
csv
FundOfFunds,MarketValue,ChildFunds
STRATEGIC_2025,1000000000,"VFIAX,VGTSX,VBMFX"
Implementation Requirements:
python
def upload_portfolio(
    file_path: str,
    portfolio_id: str,
    portfolio_type: str = "auto_detect"
) -> Dict:
    """
    Universal portfolio upload function
    
    Args:
        file_path: Path to CSV file
        portfolio_id: Unique portfolio identifier
        portfolio_type: Type of portfolio structure
            Options: "auto_detect", "individual_securities", "fund", 
                     "fund_of_funds", "mixed"
    
    Returns:
        Dict with comprehensive status and created entities
        {
            'status': 'success',
            'portfolio_id': str,
            'securities': List[Company],
            'summary': {
                'total_count': int,
                'by_type': Dict[str, int],
                'total_market_value': float,
                'errors': List[str]
            }
        }
    
    Your implementation must:
    1. Auto-detect the CSV format and identifier types
    2. Handle backward compatibility with existing equity CSVs
    3. Process bonds using the new fetch_bond_data() function
    4. Handle fund structures by resolving underlying holdings
    5. Support recursive fund of funds structures
    6. Provide detailed error handling and reporting
    7. Maintain performance for large files (1000+ securities)
    """
    # YOUR IMPLEMENTATION HERE
    pass


def fetch_bond_data(identifier: str, identifier_type: str) -> Dict:
    """
    Fetch bond data from Terrapin API (mock for this assignment)
    
    Args:
        identifier: CUSIP or ISIN
        identifier_type: "cusip" or "isin"
    
    Returns:
        Dict with bond details
    """
    # YOUR IMPLEMENTATION HERE
    # Mock Terrapin API response
    pass


def fetch_fund_holdings(fund_identifier: str) -> List[str]:
    """
    Fetch underlying holdings for a fund
    
    Args:
        fund_identifier: Fund ticker or identifier
    
    Returns:
        List of identifiers for underlying securities
    """
    # YOUR IMPLEMENTATION HERE
    pass


def resolve_fund_of_funds(fund_id: str, depth: int = 0, max_depth: int = 5) -> List[Dict]:
    """
    Recursively resolve fund of funds structure to individual securities
    
    Args:
        fund_id: Fund identifier
        depth: Current recursion depth
        max_depth: Maximum recursion depth (prevent infinite loops)
    
    Returns:
        List of individual securities with weights
    """
    # YOUR IMPLEMENTATION HERE
    pass
Design Considerations:
Column Name Flexibility: The function should accept various column names:
Ticker, Symbol, Identifier for equity
CUSIP, Identifier for bonds
ISIN, Identifier for sovereign bonds
Error Handling: Gracefully handle:
Missing columns
Invalid identifiers
API failures
Malformed CSV files
Circular fund references
Performance: Optimize for:
Batch API calls (don't call API for each security individually)
Caching (avoid redundant API calls for same security)
Memory efficiency (handle large files)
Extensibility: Make it easy to add:
New asset types (derivatives, crypto, etc.)
New identifier types
New data sources
Task 4: Portfolio Metrics Calculation (1 hour)
Objective: Implement portfolio-level metrics that work across all structures.

python
def calculate_portfolio_metrics(portfolio_id: str) -> Dict:
    """
    Calculate portfolio-level metrics regardless of structure
    
    Should handle:
    - Individual securities portfolio: Direct aggregation
    - Fund portfolio: Look-through to underlying holdings
    - Fund of Funds: Recursive look-through to ultimate holdings
    - Mixed portfolio: Weighted combination
    
    Metrics to calculate:
    1. Total market value
    2. Asset allocation (equity %, bond %, etc.)
    3. Weighted average carbon intensity
    4. Portfolio climate score
    5. Top 10 holdings (resolved to individual securities)
    6. Geographic distribution (if data available)
    7. Sector allocation (for equities)
    8. Credit rating distribution (for bonds)
    
    Returns:
        {
            'portfolio_id': str,
            'total_market_value': float,
            'asset_allocation': {
                'equity': float,  # percentage
                'municipal_bond': float,
                'sovereign_bond': float
            },
            'climate_metrics': {
                'weighted_carbon_intensity': float,
                'portfolio_climate_score': float
            },
            'top_holdings': [
                {
                    'identifier': str,
                    'name': str,
                    'weight': float,
                    'market_value': float
                }
            ],
            'bond_metrics': {
                'weighted_duration': float,
                'credit_rating_distribution': Dict[str, float]
            }
        }
    """
    # YOUR IMPLEMENTATION HERE
    pass


def calculate_weighted_metric(securities: List[Company], metric_name: str) -> float:
    """
    Calculate market-value weighted average for any metric
    
    Examples:
        - Weighted carbon intensity
        - Weighted duration
        - Weighted P/E ratio
    """
    # YOUR IMPLEMENTATION HERE
    pass
Task 5: Testing & Documentation (30 minutes)
Required Test Cases:
Create comprehensive tests in test_portfolio_service.py:

python
import pytest
from portfolio_service import upload_portfolio, calculate_portfolio_metrics

class TestBackwardCompatibility:
    """Ensure existing equity functionality still works"""
    
    def test_existing_equity_csv_format(self):
        """Original equity CSV format must work unchanged"""
        result = upload_portfolio('test_data/equity_legacy.csv', 'PORT001')
        assert result['status'] == 'success'
        assert all(c.security_type == 'equity' for c in result['securities'])
    
    def test_existing_ticker_column_name(self):
        """Must accept 'Ticker' column name"""
        pass
    
    def test_climate_metrics_still_calculated(self):
        """Existing climate features must continue working"""
        pass


class TestBondSupport:
    """Test new bond functionality"""
    
    def test_cusip_detection(self):
        """CUSIP identifiers should be detected as municipal bonds"""
        pass
    
    def test_isin_detection(self):
        """ISIN identifiers should be detected as sovereign bonds"""
        pass
    
    def test_bond_only_upload(self):
        """Upload CSV containing only bonds"""
        pass
    
    def test_bond_data_fetching(self):
        """Bond-specific data should be fetched and stored"""
        pass


class TestMixedPortfolios:
    """Test mixed asset portfolios"""
    
    def test_equity_bond_mix(self):
        """Upload CSV with both equities and bonds"""
        pass
    
    def test_generic_identifier_column(self):
        """Handle 'Identifier' column with auto-detection"""
        pass


class TestFundSupport:
    """Test fund and fund of funds"""
    
    def test_fund_portfolio(self):
        """Upload fund portfolio and resolve holdings"""
        pass
    
    def test_fund_of_funds(self):
        """Recursively resolve fund of funds structure"""
        pass
    
    def test_circular_reference_detection(self):
        """Detect and handle circular fund references"""
        pass


class TestMetricsCalculation:
    """Test portfolio metrics"""
    
    def test_asset_allocation_calculation(self):
        """Correctly calculate asset allocation percentages"""
        pass
    
    def test_weighted_carbon_intensity(self):
        """Calculate weighted metrics across mixed portfolio"""
        pass
    
    def test_fund_lookthrough_metrics(self):
        """Metrics should look through funds to underlying securities"""
        pass


class TestErrorHandling:
    """Test error handling and edge cases"""
    
    def test_invalid_csv_format(self):
        """Gracefully handle malformed CSV"""
        pass
    
    def test_missing_columns(self):
        """Provide clear error for missing required columns"""
        pass
    
    def test_invalid_identifier(self):
        """Handle invalid/unknown identifiers"""
        pass
    
    def test_api_failure_resilience(self):
        """Continue processing when API calls fail"""
        pass
Documentation Requirements:
Create detailed documentation in your submission:

IMPLEMENTATION.md:
Your design approach and architecture decisions
How you maintained backward compatibility
Trade-offs you considered
Performance optimizations
Future extensibility considerations
API_DOCUMENTATION.md:
Complete function signatures
Parameter descriptions
Return value specifications
Usage examples
Error codes and handling
SETUP.md:
Installation instructions
Dependencies
Database setup
Running tests
Deployment considerations
🏆 Evaluation Criteria
Your submission will be evaluated on:

Criterion	Weight	What We're Looking For
Backward Compatibility	25%	Existing equity CSV files work unchanged<br>No breaking changes to current functionality<br>Existing portfolios load correctly
Code Design	25%	Clean abstractions and separation of concerns<br>DRY principle (no unnecessary duplication)<br>Easy to extend for new asset types<br>Appropriate use of design patterns
Correctness	20%	All test cases pass<br>Handles edge cases properly<br>Accurate calculations<br>Proper data validation
Error Handling	15%	Graceful degradation<br>Informative error messages<br>No silent failures<br>Proper exception handling
Performance	10%	Efficient for large files (1000+ securities)<br>Minimal redundant API calls<br>Smart caching strategy<br>Reasonable memory usage
Documentation	5%	Clear code comments<br>Comprehensive README<br>API documentation<br>Design decisions explained
📊 Test Data
The repository includes test data files in the test_data/ directory:

equity_legacy.csv (Backward Compatibility Test)
csv
Ticker,MarketValue
AAPL,150000000
MSFT,120000000
GOOGL,180000000
AMZN,140000000
TSLA,110000000
bonds_municipal.csv
csv
CUSIP,MarketValue
037833100,100000000
313369AJ3,80000000
64966LAA4,75000000
bonds_sovereign.csv
csv
ISIN,MarketValue
US0378331005,150000000
US91282CHX4,120000000
US459200HU15,100000000
mixed_portfolio.csv
csv
Identifier,MarketValue
AAPL,150000000
037833100,100000000
MSFT,120000000
US91282CHX4,90000000
GOOGL,130000000
313369AJ3,70000000
fund_portfolio.csv
csv
FundIdentifier,MarketValue,Holdings
VFIAX,500000000,"AAPL,MSFT,GOOGL,JPM,JNJ,V,PG,UNH,HD,DIS"
VGTSX,300000000,"TSM,ASML,SAP,TM,HSBC"
VBMFX,400000000,"037833100,313369AJ3,64966LAA4"
fund_of_funds.csv
csv
FundOfFunds,MarketValue,ChildFunds
STRATEGIC_2025,1000000000,"VFIAX,VGTSX,VBMFX"
BALANCED_2030,800000000,"VFIAX,VBMFX"
edge_cases.csv (Malformed for Error Testing)
csv
Identifier,MarketValue
AAPL,not_a_number
INVALID_TICKER,100000
,50000000
037833100,
📤 Submission Guidelines
What to Submit:
Code Files:
models.py (extended)
portfolio_service.py (extended)
migration_001_add_bond_support.sql
test_portfolio_service.py
Any additional helper modules you create
Documentation:
IMPLEMENTATION.md (your design approach)
API_DOCUMENTATION.md (function specifications)
SETUP.md (installation and testing instructions)
Tests:
Complete test suite with all required test cases
Test coverage report (if possible)
How to Submit:
Option 1: Fork this Repository (Preferred)
Fork this repository to your GitHub account
Create a new branch: git checkout -b solution
Implement your solution
Commit with clear messages
Push to your fork
Share the repository link with us
Option 2: Zip File
Complete your solution
Include all files mentioned above
Create a zip file: portfolio_assignment_[your_name].zip
Email to: [recruiter_email@company.com]
Submission Checklist:
 All existing equity tests pass
 New bond tests pass
 Mixed portfolio tests pass
 Fund and fund of funds tests pass
 Error handling tests pass
 Migration script included
 Documentation complete
 Code is well-commented
 README explains how to run everything
 No sensitive data or API keys in code
❓ FAQ
Q: Can I use external libraries?
A: Yes, but justify your choices. Stick to common Python libraries (pandas, SQLAlchemy, pytest). Avoid over-engineering with unnecessary dependencies.

Q: Should I implement actual API calls?
A: No, use mock implementations. Focus on the architecture and data flow. Your mocks should return realistic data structures.

Q: How much error handling is expected?
A: Be thorough but not paranoid. Handle realistic error scenarios (malformed files, missing data, API failures) with clear, actionable error messages.

Q: Can I change the existing Company model structure?
A: You can ADD columns and indexes, but you cannot REMOVE or RENAME existing columns. The model must remain backward compatible.

Q: What if I don't finish everything?
A: Prioritize in this order:

Backward compatibility (critical)
Basic bond support (high priority)
Mixed portfolios (high priority)
Fund support (medium priority)
Fund of funds (nice to have)
Submit what you have with clear documentation of what's incomplete and how you'd complete it.

Q: Should I optimize for performance from the start?
A: Start with correctness, then optimize. Document any performance considerations and trade-offs you made.

Q: How should I handle funds that contain other funds?
A: Implement recursive resolution with a max depth limit (prevent infinite loops). Consider caching to avoid redundant lookups.

Q: What's more important: clever code or readable code?
A: Readable code wins every time. We value maintainability over cleverness.

💡 Hints for Success
Start with This Approach:
Phase 1: Schema (30 min)
Extend the Company model
Write and test the migration
Ensure existing data remains valid
Phase 2: Detection (15 min)
Implement detect_security_type()
Test with various identifiers
Handle edge cases
Phase 3: Simple Extension (1 hour)
Make existing upload function handle bonds
Keep equity logic unchanged
Test backward compatibility
Phase 4: Mixed Portfolios (1 hour)
Add auto-detection of CSV format
Handle mixed asset files
Comprehensive error handling
Phase 5: Funds (1-2 hours)
Implement fund resolution
Add fund of funds support
Test recursive structures
Phase 6: Metrics & Polish (30 min)
Implement portfolio metrics
Final testing
Documentation
Technical Tips:
Use pandas effectively: DataFrames make CSV handling much easier
Cache API results: Don't fetch the same security twice
Batch operations: Group securities by type for efficient processing
Validate early: Check CSV format before processing all rows
Think about transaction boundaries: What happens if upload fails halfway?
Consider idempotency: What if the same file is uploaded twice?
Common Pitfalls to Avoid:
❌ Breaking existing equity functionality
❌ Creating separate tables for each asset type
❌ Not handling edge cases (empty strings, whitespace, null values)
❌ Silent failures (always report errors clearly)
❌ Over-engineering (keep it simple and extensible)
❌ No error messages (users need to know what went wrong)
❌ Forgetting about performance (large files should work)
🤔 Thought-Provoking Questions
After completing this assignment, you should be able to answer:

Architecture:
Why extend one table vs. creating separate tables per asset type?
How would you handle 100 different asset types in the future?
What are the trade-offs of your chosen approach?
Data Flow:
How does your solution handle a fund containing both equities and bonds?
What happens if a Fund of Funds has circular references (Fund A → Fund B → Fund A)?
How would you handle partial failures (some securities succeed, others fail)?
Performance:
Your system needs to process a CSV with 10,000 securities. How long should it take?
How would you optimize API calls to external data providers?
What's your caching strategy?
Extensibility:
How would you add support for derivatives (options, futures)?
What if the business wants to add crypto assets?
How would you add a new data field to all asset types?
Error Handling:
A CSV has 1000 rows, and row 500 has an invalid identifier. What should happen?
The bond API is down. Should the entire upload fail?
How do you communicate errors to users in a helpful way?
Real-World Scenarios:
A fund updates its holdings daily. How do you keep data fresh?
Two portfolios contain the same securities. How do you avoid redundant storage?
Regulatory requirements change and you need to add new compliance fields. How?
🎓 What We're Really Testing
Beyond the code, we want to see:

Problem-solving approach: Do you break down complex problems systematically?
Design thinking: Do you consider trade-offs and future extensibility?
Attention to detail: Do you handle edge cases and validate inputs?
Communication: Are your design decisions clear and well-documented?
Pragmatism: Do you balance perfection with practical delivery?
Learning ability: Can you understand and extend an existing codebase?
📞 Questions?
If you have questions about the assignment:

Technical clarifications: Email [tech-hiring@company.com]
Submission issues: Email [recruiter@company.com]
Timeline extensions: Contact your recruiter
We typically respond within 24 hours on business days.

⏰ Timeline
Assignment sent: [Date]
Submission deadline: [Date + 1 week]
Review period: 3-5 business days
Technical discussion: If selected, we'll schedule a call to discuss your solution
🚀 Good Luck!
This assignment reflects real challenges you'll tackle in this role. We're excited to see your approach!

Remember:

✅ Backward compatibility is critical
✅ Clean, readable code over clever tricks
✅ Document your design decisions
✅ Test thoroughly
✅ Have fun with it!
We look forward to reviewing your solution! 🎉

Repository Structure:

portfolio-assignment/
├── README.md (this file)
├── models.py (starter code)
├── portfolio_service.py (starter code)
├── test_data/
│   ├── equity_legacy.csv
│   ├── bonds_municipal.csv
│   ├── bonds_sovereign.csv
│   ├── mixed_portfolio.csv
│   ├── fund_portfolio.csv
│   ├── fund_of_funds.csv
│   └── edge_cases.csv
├── tests/
│   └── README.md (testing guidelines)
└── docs/
    └── ARCHITECTURE_GUIDE.md (additional context)
