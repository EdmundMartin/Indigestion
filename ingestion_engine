import re
from queue import Queue
from concurrent.futures import ThreadPoolExecutor
import requests
from bs4 import BeautifulSoup
import logging


class Ingestion:

    def __init__(self, sitemap, regex, concurrency=20, proxy=None):
        self.regex = re.compile(regex)
        self.links = set([])
        self.queue = Queue()
        self.queue.put(sitemap)
        self.thread_pool = ThreadPoolExecutor(max_workers=concurrency)
        self.user_agent = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.186 Safari/537.36'
        self.proxy = {'https': proxy, 'http': proxy}

    def parser(self, response):
        raise NotImplementedError

    def _parser_callback(self, item):
        item = item.result()
        if item:
            self.parser(item)

    def _recursive_sitemap_parse(self):
        while not self.queue.empty():
            sitemap = self.queue.get_nowait()
            self.__parse_sitemap_xml(sitemap)

    def __parse_sitemap_xml(self, sitemap):
        try:
            response = requests.get(sitemap)
            soup = BeautifulSoup(response.text, 'lxml-xml')
            raw_urls = soup.find_all('loc')
            for raw in raw_urls:
                if '.xml' in raw.text:
                    self.queue.put(raw.text)
                elif re.match(self.regex, raw.text):
                    self.links.add(raw.text)
        except Exception as e:
            logging.warning("Exception retrieving sitemap: {}, Exception: {}".format(sitemap, e))

    def _get_response_object(self, url):
        try:
            response = requests.get(url, proxies=self.proxy, timeout=(30, 60))
            return response
        except requests.RequestException as e:
            return

    def digest(self):
        self._recursive_sitemap_parse()
        for link in self.links:
            t = self.thread_pool.submit(self._get_response_object, link)
            t.add_done_callback(self._parser_callback)
