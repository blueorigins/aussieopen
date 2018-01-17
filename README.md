import bs4
import requests
import csv
import boto3
import os

def getmatches():
    soup = bs4.BeautifulSoup(requests.get('https://ausopen.com/scores/live').text, 'html.parser')
    players = soup.find_all('div',{'class':'field field--name-field-short-name field--type-string field--label-hidden field__item'})
    live = []
    for player in players:
        live.append(player.text)
    newmatch(live)

def newmatch(curmatch):
    existing = []
    with open('/Users/beckwith/Desktop/python/beckwith/tennis.csv','r') as f:
        reader = csv.reader(f)
        for row in reader:
            existing.append(row)
    
    new_match = list(set(curmatch) - set(existing[0]))
    store(curmatch)
    if len(new_match) > 0:
        send_notification(new_match)

def store(matches):
    with open('tennis.csv','w') as f:
        write = csv.writer(f)
        write.writerow(matches)

def send_notification(players):
    if 'R. Federer' in players:
        tos3('Federer')
    elif 'G. Monfils' in players:
        tos3('Monfils')

def tos3(player):
    s3 = boto3.resource('s3')
    s3.Bucket('tennismatches').put_object(Key='{}.txt'.format(player))
    print('Great Success')

if __name__ == "__main__":
    getmatches()
