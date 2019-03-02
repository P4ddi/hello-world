# modules#############################################################################################
import requests
import json
import time
import http.client, urllib.request, urllib.parse, urllib.error, base64
import datetime
# paramters#############################################################################################
height = 4.0
am1 = datetime.time(8,00,0) # morning starts
am2 = datetime.time(13,00,0) # afternoon starts
pm1 = datetime.time(17,00,0) # evening starts
pm2 = datetime.time(21,00,0) # night starts
height = 4.0
wd1 = 0 # wind direction lower bound
wd2 = 90 # wind direction upper bound
ws1 = 15 # wind speed lower bound
ws2 = 30 # wind speed upper bound
wg1 = 35 # wind gust upper bound
wc1 = ['4', '5', '6', '7', '8', '9', '13', '18', '23', '24', '25', '26', '27', '28', '29', '33', '34', '35','36']  # weather codes - use as a stop list ie. "weather not in wc1"
#getPeriod function#######################################################################################
def getPeriod(api):
    if (am1 < api < am2):
        period = '1'
    elif (am2 < api < pm1):
        period = '2'
    elif (pm1 < api < pm2):
        period = '3'
    else:
        period = 'night'
    return period
# ADMIRALTY TIDE DATA ##################################################################################
headers = {'Ocp-Apim-Subscription-Key': 'b8e46b3f49014df090ee85bf270825cc',}
params = urllib.parse.urlencode({'duration': '7',})
try:
    conn = http.client.HTTPSConnection('admiraltyapi.azure-api.net')
    conn.request("GET", "/uktidalapi/api/V1/Stations/0068/TidalEvents?%s" % params, "{body}", headers)
    response = conn.getresponse()
    tidedata = response.read()
    my_json2 = tidedata.decode('utf8').replace("'", '"')
    data2 = json.loads(my_json2)
    tide_result=[]
    for j in range(len(data2)):
        api2 = datetime.datetime.strptime(data2[j]['DateTime'], '%Y-%m-%dT%H:%M:%S').time()
        api1 = datetime.datetime.strptime(data2[j]['DateTime'], '%Y-%m-%dT%H:%M:%S').date()
        period = getPeriod(api2)
        if data2[j]['Height'] < height and period != 'night':
            tide_result.append(str(api1)+' '+period)
    conn.close()
except Exception as e:
    print("[Errno {0}] {1}".format(e.errno, e.strerror))
# MAGIC SEAWEED WEATHER DATA###########################################################################
response = requests.get("http://magicseaweed.com/api/88391fc1284a774e73ed4914bbd0f609/forecast/?spot_id=3765")
x = response.content
my_json = x.decode('utf8').replace("'", '"')
data = json.loads(my_json)
s = json.dumps(data, indent=4, sort_keys=True)
#print(time.ctime(data[0]['localTimestamp']))
weather_result = []
for i in range(len(data)):
    api3 = datetime.datetime.utcfromtimestamp(data[i]['localTimestamp']).date()
    api4 = datetime.datetime.utcfromtimestamp(data[i]['localTimestamp']).time()
    period = getPeriod(api4)
    if ws1 <= data[i]['wind']['speed'] <= ws2 and wd1 <= data[i]['wind']['direction'] <= wd2 and data[i]['wind']['gusts'] <= wg1 and period != 'night' and data[i]['condition']['weather'] not in wc1:
        weather_result.append(str(api3) + ' ' + period)
#results######################################################################################################
matches = sorted(set(tide_result).intersection(weather_result))
print('you can kite on: ', matches)
# fetch weather icons##################################################################################
# for i in range(100):
#     url = ('http://cdnimages.magicseaweed.com/30x30/%s.png' %(i))
#     response = requests.get(url)
#     if response.status_code == 200:
#         filename = ('/Users/andrew.paddison/Desktop/pics/%s.png' %(i))
#         with open(filename, 'wb') as f:
#             f.write(response.content)
######################################################################################################






