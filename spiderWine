#!/usr/bin/python
#-*- coding: utf-8 -*-
from ProjetVinNicolas3.items import Projetvinnicolas3Item
from scrapy.contrib.exporter import CsvItemExporter
from scrapy.contrib.linkextractors.sgml import SgmlLinkExtractor
from scrapy.contrib.spiders import CrawlSpider, Rule
from scrapy.selector import HtmlXPathSelector
from scrapy.contrib.pipeline.images import ImagesPipeline
import lxml
import lxml.etree
import lxml.html
import re
import urlparse
#<a href="javascript:val('/page.php/fr/18_409~80~10.htm')" class="glo_pagination_link_page">9</a>
def strip_javascript(value):
    m = re.search("javascript:val\('(\S+~(\d+)~10\.htm).*'\)", value)
    if m:
        return m.group(1)

class MySpider(CrawlSpider):
    name = "vinNico3"
    allowed_domains = ["nicolas.com"]
    start_urls = ["http://www.nicolas.com/fr/commander_bordeaux.html"]
    # la regle pour suivre les liens des pages suivantes  
    rules = (
        Rule(
            SgmlLinkExtractor(
                restrict_xpaths='//div[@class="glo_pagination_droite"]/a[1]',
                process_value=strip_javascript,
                unique=True
            ),
            follow=True,
        ),
        Rule(
            SgmlLinkExtractor(
    # fixer les liens vers chaque bouteille du vin 
                restrict_xpaths='//table[@class="cpt_fav_table_commande"]/tr[position()>1]/td[3]/a[1]',
            ),
            callback='parse_wine_page', follow=True,
        ),
    )
    #Fonction qui extrait les donn�es � partir de chaque lien d'une bouteille du vin 
    def parse_wine_page(self, response):
        items = []
        item = Projetvinnicolas3Item()
        # lxml only for temperature de conservation 
        root = lxml.html.fromstring(response.body)
        for content in root.xpath('//*[@id="glo_right"]'):
           temperature = content.xpath('.//div[@class="pro_col_right"]/div[@class="pro_blk_trans"]/br')
           if temperature:
               #print 'OK Température'
               item ["temp_Conserv"] = lxml.html.tostring(temperature[0], method="text", encoding=unicode).strip()
        # back to hxs 
        hxs = HtmlXPathSelector(response)
        content = hxs.select('//*[@id="glo_right"]')
        for res in content:
            #Méthode 1
            #title = map(unicode.strip,
            #    content.select('.//div[@class="pro_detail_tit"]/div[@class="pro_titre"]/h1/text()').extract())
            #m = re.compile(r'^(?P<nomVin1>.+)\s+(?P<millesime>\d+)$').search(title[0])
            #if m:
            #    item ["nomVin1"]=m.group("nomVin")
            #    item ["millesime"]=m.group("millesime")
            
            #Méthode 2 
            nomVin1, millesime = res.select(
               './/div[@class="pro_detail_tit"]/div[@class="pro_titre"]/h1/text()').re(
                   r'^(.+)\s+(\d+)\s*$')
            nomVin1=nomVin1.strip()
            item["millesime"]=millesime.strip()
            iNomVin= map(unicode.strip, res.select('//div[@class="pro_detail_tit"]//div[@class="pro_titre"]/h1/text()').extract())
            #print type(x)
            iNomVin=(str(iNomVin[0].encode('utf-8'))).lower()
            item ["nomVin"]=iNomVin.strip()
            
            #Methode 1
            item ["classement"] = map(unicode.strip, res.select('.//div[@class="pro_col_right"]/div[@class="pro_blk_trans"]/div[@class="pro_blk_trans_titre"]/text()').re(r'^(\d\w+\s*\w+)'))
            #iClassement=(str(iClassement[0].encode('utf-8'))).lower()
            #item ["classement"]=iClassement.strip()            
            iAppelation = map(unicode.strip, res.select('.//div[@class="pro_col_right"]/div[@class="pro_blk_trans"]/div[@class="pro_blk_trans_titre"]/text()').re(r'([\w-]+)(?=\W+(Rouge|Blanc|Rosé))'))
            iAppelation=(str(iAppelation[0].encode('utf-8'))).lower()
            item ["appelation"] =iAppelation.strip()
            
            iCouleur = map(unicode.strip, res.select('.//div[@class="pro_col_right"]/div[@class="pro_blk_trans"]/div[@class="pro_blk_trans_titre"]/text()').re(r'\w+\s*$'))
            iCouleur =(str(iCouleur[0].encode('utf-8'))).lower()
            item ["couleur"] =iCouleur.strip()
            #Methode 2
            #title = map(unicode.strip,
            #    content.select('.//div[@class="pro_col_right"]/div[@class="pro_blk_trans"]/div[@class="pro_blk_trans_titre"]/text()').extract())
            #m = re.compile(r'^(?P<classement>\d\w+\s*\w+)\S\s+(?P<appelation>\w+\-\w+|\w+)\S\s+(?P<couleur>\w+)\s*$').search(title[0])
            #if m:
            #    item ["classement"]=m.group("classement")
            #    item ["appelation"]=m.group("appelation")
            #    item ["couleur"]=m.group("couleur")
            
            verifCepage=map(unicode.strip, res.select('//div[@class="pro_col_right"]/div[@class="pro_blk_mauve"]/div[1]/text()').extract())
            if 'page' in str(verifCepage) :
                item ["cepage_s"] = map(unicode.strip, res.select('//div[@class="pro_col_right"]/div[@class="pro_blk_mauve"]/div[2]/text()').re(r'\d+%\s+.+'))
                #iCepage_s=(str(iCepage_s[0].encode('utf-8'))).lower()
                #item ["cepage_s"] = iCepage_s.strip()
            else:
                item ["cepage_s"] = ''
            item ["temperature_de_service"] = map(unicode.strip, res.select('//div[@class="pro_col_right"]//div[@class="pro_blk_mauve"]//div[@class="pro_lgn_contenu"]/text()').re(r'\d+[°]'))
            item ["volume"]= map(unicode.strip, res.select('//div[@class="pro_col_right"]/table[@class="pro_table_commande"]/tr/td[@class="pro_td_commande_1"]/div[@class="pro_commande_prix_3"]/text()').re(r'\d+\scl'))
            item ["prix"]= map(unicode.strip, res.select('//div[@class="pro_col_right"]/table[@class="pro_table_commande"]/tr/td[@class="pro_td_commande_1"]/div[@class="pro_commande_prix"]/text()').re(r'\d+.\d*'))
            #item ["Image"]= map(unicode.strip, res.select('//div[@class="pro_detail_tit"]//div[@class="pro_titre"]/h1/text()').extract())
            #link = res.select('//div[@class="pro_col_left"]//img/@src').extract()
            #link=str(link)
            item['image_urls'] = map(lambda src: urlparse.urljoin(response.url, src), res.select('./div[@class="pro_col_left"]/img/@src').extract())
        items.append(item)
        return items
        
    #parse_start_url = parse_wine_page
