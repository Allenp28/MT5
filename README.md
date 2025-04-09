# ICM
import pandas as pd

from icm import ICMClient
from icm.utils import miliseconds_since_epoch

from datetime import datetime
from datetime import time
from datetime import timezone

from typing import List
from typing import Dict
from typing import Union

from pyrobot.portfolio import Portfolio


class PyRobot:


    def __init__ (self, client_id: str, redirect_url: str, credentials_path: str = None, trading_account: str = None, paper_trading: bool = True) -> None:

      self.trading_account: str = trading_account  
      self.client_id: str = client_id
      self.redirect_uri: str = redirect_url
      self.credentials_path: str = credentials_path
      self.session: ICMClient = self._create_session()
      self.trades: dict = {}
      self.historical_prices: dict = {}
      self.stock_frame = None
      self.paper_trading = paper_trading

    def _create_session(self) -> ICMClient: 
      
        icm_client = ICMClient(
                client_id=self.client_id,
                redirect_url=self.redirect_uri,
                credentials_path=self.credentials_path
        )

        # login to the session
        icm_client.login()

        return icm_client
    
    @property
    def pre_market_open(self) -> bool:
        
        pre_market_start_time = datetime.now().replace(hour=12, minute=00, second=00, tzinfo=timezone.utc).timestamp()
        market_start_time = datetime.now().replace(hour=13, minute=30, second=00, tzinfo=timezone.utc).timestamp()
        right_now = datetime.now().replace(tzinfo=timezone.utc).timestamp()

        if market_start_time >= right_now >=pre_market_start_time:

            return True
        else:
            return False

    @property
    def post_market_open(self) -> bool:
         
         post_market_end_time = datetime.now().replace(hour=22, minute=30, second=00, tzinfo=timezone.utc).timestamp()
         market_end_time = datetime.now().replace(hour=20, minute=00, second=00, tzinfo=timezone.utc).timestamp()
         right_now = datetime.now().replace(tzinfo=timezone.utc).timestamp()

         if post_market_end_time >= right_now >= market_end_time:
            return True
         else:
            return False

    @property
    def regular_market_open(self) -> bool:
         
         market_start_time = datetime.now().replace(hour=13, minute=30, second=00, tzinfo=timezone.utc).timestamp()
         market_end_time = datetime.now().replace(hour=20, minute=00, second=00, tzinfo=timezone.utc).timestamp()
         right_now = datetime.now().replace(tzinfo=timezone.utc).timestamp()

         if market_end_time >= right_now >= market_start_time:
            return True
         else:
            return False

    def create_portfolio(self):

        # Initialize a new Portfolio object.
        self.portfolio = Portfolio(account_number=self.trading_account)

        # Assign the client.
        self.portfolio._icm_client = self.session

        return self.portfolio


    def create_trade(self):
        pass

    def create_stack_frame(self):
        pass

    def grab_current_quotes(self) ->dict:
        
        # First grab all the symbols.
        symbols = self.portfolio.positions.key()

        # Grab the quotes.
        quotes = self.session.get_quotes(instrument=list(symbols))
        
        return quotes
    
    def grab_historical_prices(self) -> List[Dict]:
        pass

    
