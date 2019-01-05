"""
digiKeyPartGrabber.py: This is a simple object that is uses requests to gather specific data about a given Digikey part
number.  The current version is very limited and only parsing a small number of values about a part. Testing on this
is limited to the components that I actually needed.

"""
import requests
import bs4
import re


# Define a handful of Exceptions
class DigikeyReturnError(Exception):
    """ Passed when the part request doesn't return successfully """
    pass


class DigikeyWrongPartNumber(Exception):
    """ Passed when the request doesn't return expected part """
    pass


class DigikeyNoPartIdentified(Exception):
    """ Passed when a part ID is not found in the request content """
    pass


class DigikeyPart(object):
    """
    DigikeyPart: This object manages the data request from Digikey.com and parses the data into member variables.
    """

    headers = {'user-agent': 'Chrome/71.0.3578.98 Safari/537.36',
               'Accept-Encoding': 'gzip, deflate', 'Accept': '*/*',
               'Connection': 'keep-alive',
               'Referrer Policy': 'no-referrer-when-downgrade',
               'upgrade-insecure-requests': '1',
               'cookie': '0'}

    ohmMultiplier = {'ohms': 1, 'kohms': 1e3, 'mohms': 1e6}  # Lower cases, just is case...
    capMultiplier = {'pf': 1e-12, 'nf': 1e-9, 'µf': 1e-6}  # Lower cases, just is case...

    def __init__(self, partNum):
        """
        Instantiate the DigikeyPart object, grab the product web page, and

        :param partNum: (String) A Digikey Part Number
        :return: None
        """
        self.partNum = partNum
        self.model = None
        self.manufacturer = None
        self.description = None
        self.value = None
        self.package = None
        self.power = None
        self.tolerance = None

        # Grab the product webpage, raise an exception if there is an issues otherwise, parse the data
        payload = {'keywords': partNum}
        r = requests.get('https://www.digikey.com/products/en', params=payload, headers=self.headers)
        if r.status_code != 200:
            raise DigikeyReturnError
        self.parseResponse(r.content)

    def parseResponse(self, content):
        """
        parseResponse: Given the contents of a Didgikey product webpage, parse out the data of interest

        :param content: The contents of of the a Digikey product web page
        :return: None
        """
        # Ensure that the content contains product information
        soup = bs4.BeautifulSoup(content, features="html.parser")
        metaTag = soup.find("meta", itemprop='productID')
        if metaTag is None:
            raise DigikeyNoPartIdentified

        # Ensure that the content contains the details for the right part #
        partNum = metaTag.attrs['content'].split(':')[-1]
        if partNum != self.partNum:
            raise DigikeyWrongPartNumber

        # Parse all of the information we expect for all of the parts
        self.model = soup.find("h1", itemprop='model').text.strip()
        self.manufacturer = soup.find("span", itemprop='name').text.strip()
        self.description = soup.find("td", itemprop='description').text.strip()
        self.package = soup.find("th", text=re.compile("Package")).find_next().text.strip().split(' ')[0]

        # Determine if the part is a resistor; if so, parse resistor specific values
        resistor = soup.find("th", text=re.compile("Resistance"))
        if resistor is not None:
            self.value = self.parseResistance(resistor.find_next().text.strip())
            self.tolerance = soup.find("th", text=re.compile("Tolerance")).find_next().text.strip()
            self.power = soup.find("th", text=re.compile("Power")).find_next().text.strip().split(',')[0]

        # Determine if the part is a capacitor; if so, parse capacitor specific values
        capacitor = soup.find("th", text=re.compile("Capacitance"))
        if capacitor is not None:
            self.value = self.parseCapacitance(capacitor.find_next().text.strip())
            self.tolerance = soup.find("th", text=re.compile("Tolerance")).find_next().text.strip()

    @classmethod
    def parseResistance(cls, resistance):
        """
        parseResistance: Takes the string from resistance string and converts it to a number. For example, 1 kOhms
        becomes 1000

        :param resistance: A string containing the resistance value displayed on the product web page
        :return: A numerical representative of  resistance
        """
        r = re.findall(r'[A-Za-z]+|\d+', resistance)
        return int(r[0]) * cls.ohmMultiplier[r[1].lower()]

    @classmethod
    def parseCapacitance(cls, capacitance):
        """
        parseCapacitance: Takes the string from capacitance string and converts it to a number. For example, 1 kOhms
        becomes 1000

        :param capacitance: A string containing the capacitance value displayed on the product web page
        :return: A numerical representative of  capacitance
        """
        r = re.findall(r'[A-Za-z]+|\d+', capacitance)
        return int(r[0]) * cls.capMultiplier[r[1].lower()]

    def __repr__(self):
        x = (f'Digikey Part \n '
             f'    Digikey Part Number =  {self.partNum} \n '
             f'    Manufactuer Part #  =  {self.model} \n '
             f'    Manufacturer Name   =  {self.manufacturer} \n '
             f'    Description         =  {self.description} \n '
             f'    Value               =  {self.value} \n '
             f'    Power               =  {self.power} \n ')
        return x


if __name__ == '__main__':
    print(DigikeyPart('AD7779ACPZ-ND'))
