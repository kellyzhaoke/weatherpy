

```python
import random
import requests
import json
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from config import gkey
from config import wkey
from citipy import citipy
import openweathermapy.core as ow
```


```python
city = pd.DataFrame()

for x in range (550):
    ran_lats= np.random.uniform(low=-90.000, high=90.000, size=1)
    ran_lngs= np.random.uniform(low=-180.000, high=180.000, size=1)
    ran_city = pd.DataFrame([[ran_lats, ran_lngs]],columns = ["ran_lats", "ran_lngs"]).astype(float)
    city=city.append(ran_city)
new_city=city.reset_index()
new_city.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>ran_lats</th>
      <th>ran_lngs</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>78.017546</td>
      <td>-167.973521</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>67.184381</td>
      <td>124.607338</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>13.611584</td>
      <td>-35.165114</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>-86.553142</td>
      <td>-35.423050</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>-33.228237</td>
      <td>112.642712</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Find Nearby City Name
for n in range(0, len(new_city)):
    city_lat = new_city.loc[n]["ran_lats"]
    city_lngs = new_city.loc[n]["ran_lngs"]
    city = citipy.nearest_city(city_lat, city_lngs)
    new_city.set_value(n, "City Name", city.city_name.title())
    new_city.set_value(n, "Country", city.country_code.upper())
new_city1=new_city.drop("index", axis=1)
new_city1.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ran_lats</th>
      <th>ran_lngs</th>
      <th>City Name</th>
      <th>Country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>78.017546</td>
      <td>-167.973521</td>
      <td>Lavrentiya</td>
      <td>RU</td>
    </tr>
    <tr>
      <th>1</th>
      <td>67.184381</td>
      <td>124.607338</td>
      <td>Zhigansk</td>
      <td>RU</td>
    </tr>
    <tr>
      <th>2</th>
      <td>13.611584</td>
      <td>-35.165114</td>
      <td>Porto Novo</td>
      <td>CV</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-86.553142</td>
      <td>-35.423050</td>
      <td>Ushuaia</td>
      <td>AR</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-33.228237</td>
      <td>112.642712</td>
      <td>Busselton</td>
      <td>AU</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Create blank columns for necessary fields
new_city1["Lat"] = ""
new_city1["Lng"] = ""
new_city1.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ran_lats</th>
      <th>ran_lngs</th>
      <th>City Name</th>
      <th>Country</th>
      <th>Lat</th>
      <th>Lng</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>78.017546</td>
      <td>-167.973521</td>
      <td>Lavrentiya</td>
      <td>RU</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>1</th>
      <td>67.184381</td>
      <td>124.607338</td>
      <td>Zhigansk</td>
      <td>RU</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>2</th>
      <td>13.611584</td>
      <td>-35.165114</td>
      <td>Porto Novo</td>
      <td>CV</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>-86.553142</td>
      <td>-35.423050</td>
      <td>Ushuaia</td>
      <td>AR</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>4</th>
      <td>-33.228237</td>
      <td>112.642712</td>
      <td>Busselton</td>
      <td>AU</td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>




```python
# Counter
row_count = 0
```


```python
# Loop through and grab the lat/lng using Google maps
# Loop through and grab the lat/lng using Google maps
for index, row in new_city1.iterrows():
    
    # Create endpoint URL
    params = {
        "address": row["City Name"],
        "key": gkey
    }
    target_url = "https://maps.googleapis.com/maps/api/geocode/json"
    
    # Print log to ensure loop is working correctly
    print("Now retrieving city # " + str(row_count))
    #print(target_url)
    row_count += 1
    
    # Run requests to grab the JSON at the requested URL
    city_location = requests.get(target_url, params=params).json()
    
    # Append the lat/lng to the appropriate columns
    # Use try / except to skip any cities with errors
    try: 
        city_lat = city_location["results"][0]["geometry"]["location"]["lat"]
        city_lng = city_location["results"][0]["geometry"]["location"]["lng"]
        
        new_city1.set_value(index, "Lat", city_lat)
        new_city1.set_value(index, "Lng", city_lng)
        
    except:
        print("Error with city data. Skipping")
        continue

```

    Now retrieving city # 0
    Now retrieving city # 1
    Now retrieving city # 2
    Now retrieving city # 3
    Now retrieving city # 4
    Now retrieving city # 5
    Now retrieving city # 6
    Now retrieving city # 7
    Now retrieving city # 8
    Now retrieving city # 9
    Now retrieving city # 10
    Now retrieving city # 11
    Now retrieving city # 12
    Now retrieving city # 13
    Now retrieving city # 14
    Now retrieving city # 15
    Now retrieving city # 16
    Now retrieving city # 17
    Now retrieving city # 18
    Now retrieving city # 19
    Now retrieving city # 20
    Now retrieving city # 21
    Now retrieving city # 22
    Now retrieving city # 23
    Now retrieving city # 24
    Now retrieving city # 25
    Now retrieving city # 26
    Now retrieving city # 27
    Now retrieving city # 28
    Now retrieving city # 29
    Now retrieving city # 30
    Now retrieving city # 31
    Now retrieving city # 32
    Now retrieving city # 33
    Now retrieving city # 34
    Now retrieving city # 35
    Now retrieving city # 36
    Now retrieving city # 37
    Now retrieving city # 38
    Now retrieving city # 39
    Now retrieving city # 40
    Now retrieving city # 41
    Now retrieving city # 42
    Now retrieving city # 43
    Now retrieving city # 44
    Now retrieving city # 45
    Now retrieving city # 46
    Now retrieving city # 47
    Now retrieving city # 48
    Now retrieving city # 49
    Now retrieving city # 50
    Now retrieving city # 51
    Now retrieving city # 52
    Now retrieving city # 53
    Now retrieving city # 54
    Now retrieving city # 55
    Now retrieving city # 56
    Now retrieving city # 57
    Now retrieving city # 58
    Now retrieving city # 59
    Now retrieving city # 60
    Now retrieving city # 61
    Now retrieving city # 62
    Now retrieving city # 63
    Now retrieving city # 64
    Now retrieving city # 65
    Now retrieving city # 66
    Now retrieving city # 67
    Now retrieving city # 68
    Now retrieving city # 69
    Now retrieving city # 70
    Now retrieving city # 71
    Now retrieving city # 72
    Now retrieving city # 73
    Now retrieving city # 74
    Now retrieving city # 75
    Now retrieving city # 76
    Now retrieving city # 77
    Now retrieving city # 78
    Now retrieving city # 79
    Now retrieving city # 80
    Now retrieving city # 81
    Now retrieving city # 82
    Now retrieving city # 83
    Now retrieving city # 84
    Now retrieving city # 85
    Now retrieving city # 86
    Now retrieving city # 87
    Now retrieving city # 88
    Now retrieving city # 89
    Now retrieving city # 90
    Error with city data. Skipping
    Now retrieving city # 91
    Now retrieving city # 92
    Now retrieving city # 93
    Now retrieving city # 94
    Now retrieving city # 95
    Now retrieving city # 96
    Now retrieving city # 97
    Now retrieving city # 98
    Now retrieving city # 99
    Now retrieving city # 100
    Now retrieving city # 101
    Now retrieving city # 102
    Now retrieving city # 103
    Now retrieving city # 104
    Now retrieving city # 105
    Now retrieving city # 106
    Now retrieving city # 107
    Now retrieving city # 108
    Now retrieving city # 109
    Now retrieving city # 110
    Now retrieving city # 111
    Now retrieving city # 112
    Now retrieving city # 113
    Now retrieving city # 114
    Now retrieving city # 115
    Now retrieving city # 116
    Now retrieving city # 117
    Now retrieving city # 118
    Now retrieving city # 119
    Now retrieving city # 120
    Now retrieving city # 121
    Now retrieving city # 122
    Now retrieving city # 123
    Now retrieving city # 124
    Now retrieving city # 125
    Now retrieving city # 126
    Now retrieving city # 127
    Now retrieving city # 128
    Now retrieving city # 129
    Now retrieving city # 130
    Now retrieving city # 131
    Now retrieving city # 132
    Now retrieving city # 133
    Now retrieving city # 134
    Now retrieving city # 135
    Now retrieving city # 136
    Now retrieving city # 137
    Now retrieving city # 138
    Now retrieving city # 139
    Now retrieving city # 140
    Now retrieving city # 141
    Now retrieving city # 142
    Now retrieving city # 143
    Now retrieving city # 144
    Now retrieving city # 145
    Now retrieving city # 146
    Now retrieving city # 147
    Now retrieving city # 148
    Now retrieving city # 149
    Now retrieving city # 150
    Now retrieving city # 151
    Now retrieving city # 152
    Now retrieving city # 153
    Now retrieving city # 154
    Now retrieving city # 155
    Now retrieving city # 156
    Now retrieving city # 157
    Now retrieving city # 158
    Now retrieving city # 159
    Now retrieving city # 160
    Now retrieving city # 161
    Now retrieving city # 162
    Now retrieving city # 163
    Now retrieving city # 164
    Now retrieving city # 165
    Now retrieving city # 166
    Now retrieving city # 167
    Now retrieving city # 168
    Now retrieving city # 169
    Now retrieving city # 170
    Now retrieving city # 171
    Now retrieving city # 172
    Now retrieving city # 173
    Now retrieving city # 174
    Now retrieving city # 175
    Now retrieving city # 176
    Now retrieving city # 177
    Now retrieving city # 178
    Now retrieving city # 179
    Now retrieving city # 180
    Now retrieving city # 181
    Now retrieving city # 182
    Now retrieving city # 183
    Now retrieving city # 184
    Now retrieving city # 185
    Now retrieving city # 186
    Now retrieving city # 187
    Now retrieving city # 188
    Now retrieving city # 189
    Now retrieving city # 190
    Now retrieving city # 191
    Now retrieving city # 192
    Now retrieving city # 193
    Now retrieving city # 194
    Now retrieving city # 195
    Now retrieving city # 196
    Now retrieving city # 197
    Now retrieving city # 198
    Now retrieving city # 199
    Now retrieving city # 200
    Now retrieving city # 201
    Now retrieving city # 202
    Now retrieving city # 203
    Now retrieving city # 204
    Now retrieving city # 205
    Now retrieving city # 206
    Now retrieving city # 207
    Now retrieving city # 208
    Now retrieving city # 209
    Now retrieving city # 210
    Now retrieving city # 211
    Now retrieving city # 212
    Now retrieving city # 213
    Now retrieving city # 214
    Now retrieving city # 215
    Now retrieving city # 216
    Now retrieving city # 217
    Now retrieving city # 218
    Now retrieving city # 219
    Now retrieving city # 220
    Now retrieving city # 221
    Now retrieving city # 222
    Now retrieving city # 223
    Now retrieving city # 224
    Now retrieving city # 225
    Now retrieving city # 226
    Now retrieving city # 227
    Now retrieving city # 228
    Now retrieving city # 229
    Now retrieving city # 230
    Now retrieving city # 231
    Now retrieving city # 232
    Now retrieving city # 233
    Now retrieving city # 234
    Now retrieving city # 235
    Now retrieving city # 236
    Now retrieving city # 237
    Now retrieving city # 238
    Now retrieving city # 239
    Now retrieving city # 240
    Now retrieving city # 241
    Now retrieving city # 242
    Now retrieving city # 243
    Now retrieving city # 244
    Now retrieving city # 245
    Error with city data. Skipping
    Now retrieving city # 246
    Now retrieving city # 247
    Now retrieving city # 248
    Now retrieving city # 249
    Now retrieving city # 250
    Now retrieving city # 251
    Now retrieving city # 252
    Now retrieving city # 253
    Now retrieving city # 254
    Now retrieving city # 255
    Now retrieving city # 256
    Now retrieving city # 257
    Now retrieving city # 258
    Now retrieving city # 259
    Now retrieving city # 260
    Now retrieving city # 261
    Now retrieving city # 262
    Now retrieving city # 263
    Now retrieving city # 264
    Now retrieving city # 265
    Now retrieving city # 266
    Now retrieving city # 267
    Now retrieving city # 268
    Now retrieving city # 269
    Now retrieving city # 270
    Now retrieving city # 271
    Now retrieving city # 272
    Now retrieving city # 273
    Now retrieving city # 274
    Now retrieving city # 275
    Now retrieving city # 276
    Now retrieving city # 277
    Now retrieving city # 278
    Now retrieving city # 279
    Now retrieving city # 280
    Now retrieving city # 281
    Now retrieving city # 282
    Now retrieving city # 283
    Now retrieving city # 284
    Now retrieving city # 285
    Now retrieving city # 286
    Now retrieving city # 287
    Now retrieving city # 288
    Now retrieving city # 289
    Now retrieving city # 290
    Now retrieving city # 291
    Now retrieving city # 292
    Now retrieving city # 293
    Now retrieving city # 294
    Now retrieving city # 295
    Now retrieving city # 296
    Now retrieving city # 297
    Now retrieving city # 298
    Now retrieving city # 299
    Now retrieving city # 300
    Now retrieving city # 301
    Now retrieving city # 302
    Now retrieving city # 303
    Now retrieving city # 304
    Now retrieving city # 305
    Now retrieving city # 306
    Now retrieving city # 307
    Now retrieving city # 308
    Now retrieving city # 309
    Now retrieving city # 310
    Now retrieving city # 311
    Now retrieving city # 312
    Now retrieving city # 313
    Now retrieving city # 314
    Now retrieving city # 315
    Now retrieving city # 316
    Now retrieving city # 317
    Now retrieving city # 318
    Now retrieving city # 319
    Now retrieving city # 320
    Now retrieving city # 321
    Now retrieving city # 322
    Now retrieving city # 323
    Now retrieving city # 324
    Now retrieving city # 325
    Now retrieving city # 326
    Now retrieving city # 327
    Now retrieving city # 328
    Now retrieving city # 329
    Now retrieving city # 330
    Now retrieving city # 331
    Now retrieving city # 332
    Now retrieving city # 333
    Now retrieving city # 334
    Now retrieving city # 335
    Now retrieving city # 336
    Now retrieving city # 337
    Now retrieving city # 338
    Now retrieving city # 339
    Now retrieving city # 340
    Now retrieving city # 341
    Now retrieving city # 342
    Now retrieving city # 343
    Now retrieving city # 344
    Now retrieving city # 345
    Now retrieving city # 346
    Now retrieving city # 347
    Now retrieving city # 348
    Now retrieving city # 349
    Now retrieving city # 350
    Now retrieving city # 351
    Now retrieving city # 352
    Now retrieving city # 353
    Now retrieving city # 354
    Now retrieving city # 355
    Now retrieving city # 356
    Now retrieving city # 357
    Now retrieving city # 358
    Now retrieving city # 359
    Now retrieving city # 360
    Now retrieving city # 361
    Now retrieving city # 362
    Now retrieving city # 363
    Now retrieving city # 364
    Now retrieving city # 365
    Now retrieving city # 366
    Now retrieving city # 367
    Now retrieving city # 368
    Now retrieving city # 369
    Now retrieving city # 370
    Now retrieving city # 371
    Now retrieving city # 372
    Now retrieving city # 373
    Now retrieving city # 374
    Now retrieving city # 375
    Now retrieving city # 376
    Now retrieving city # 377
    Now retrieving city # 378
    Now retrieving city # 379
    Now retrieving city # 380
    Now retrieving city # 381
    Now retrieving city # 382
    Now retrieving city # 383
    Now retrieving city # 384
    Now retrieving city # 385
    Now retrieving city # 386
    Now retrieving city # 387
    Now retrieving city # 388
    Now retrieving city # 389
    Now retrieving city # 390
    Now retrieving city # 391
    Now retrieving city # 392
    Now retrieving city # 393
    Now retrieving city # 394
    Now retrieving city # 395
    Now retrieving city # 396
    Now retrieving city # 397
    Now retrieving city # 398
    Now retrieving city # 399
    Now retrieving city # 400
    Now retrieving city # 401
    Now retrieving city # 402
    Now retrieving city # 403
    Now retrieving city # 404
    Now retrieving city # 405
    Now retrieving city # 406
    Now retrieving city # 407
    Now retrieving city # 408
    Now retrieving city # 409
    Now retrieving city # 410
    Now retrieving city # 411
    Now retrieving city # 412
    Now retrieving city # 413
    Now retrieving city # 414
    Now retrieving city # 415
    Now retrieving city # 416
    Now retrieving city # 417
    Now retrieving city # 418
    Now retrieving city # 419
    Now retrieving city # 420
    Now retrieving city # 421
    Now retrieving city # 422
    Now retrieving city # 423
    Now retrieving city # 424
    Now retrieving city # 425
    Now retrieving city # 426
    Now retrieving city # 427
    Now retrieving city # 428
    Now retrieving city # 429
    Now retrieving city # 430
    Now retrieving city # 431
    Now retrieving city # 432
    Now retrieving city # 433
    Now retrieving city # 434
    Now retrieving city # 435
    Now retrieving city # 436
    Now retrieving city # 437
    Now retrieving city # 438
    Now retrieving city # 439
    Now retrieving city # 440
    Now retrieving city # 441
    Now retrieving city # 442
    Now retrieving city # 443
    Now retrieving city # 444
    Now retrieving city # 445
    Now retrieving city # 446
    Now retrieving city # 447
    Now retrieving city # 448
    Now retrieving city # 449
    Now retrieving city # 450
    Now retrieving city # 451
    Now retrieving city # 452
    Now retrieving city # 453
    Now retrieving city # 454
    Now retrieving city # 455
    Now retrieving city # 456
    Now retrieving city # 457
    Now retrieving city # 458
    Now retrieving city # 459
    Now retrieving city # 460
    Now retrieving city # 461
    Now retrieving city # 462
    Now retrieving city # 463
    Now retrieving city # 464
    Now retrieving city # 465
    Now retrieving city # 466
    Now retrieving city # 467
    Now retrieving city # 468
    Now retrieving city # 469
    Now retrieving city # 470
    Now retrieving city # 471
    Now retrieving city # 472
    Now retrieving city # 473
    Now retrieving city # 474
    Now retrieving city # 475
    Now retrieving city # 476
    Now retrieving city # 477
    Now retrieving city # 478
    Now retrieving city # 479
    Now retrieving city # 480
    Now retrieving city # 481
    Now retrieving city # 482
    Now retrieving city # 483
    Now retrieving city # 484
    Now retrieving city # 485
    Now retrieving city # 486
    Now retrieving city # 487
    Now retrieving city # 488
    Now retrieving city # 489
    Now retrieving city # 490
    Now retrieving city # 491
    Now retrieving city # 492
    Now retrieving city # 493
    Now retrieving city # 494
    Now retrieving city # 495
    Now retrieving city # 496
    Now retrieving city # 497
    Now retrieving city # 498
    Now retrieving city # 499
    Now retrieving city # 500
    Now retrieving city # 501
    Now retrieving city # 502
    Now retrieving city # 503
    Now retrieving city # 504
    Now retrieving city # 505
    Now retrieving city # 506
    Now retrieving city # 507
    Now retrieving city # 508
    Now retrieving city # 509
    Now retrieving city # 510
    Now retrieving city # 511
    Now retrieving city # 512
    Now retrieving city # 513
    Now retrieving city # 514
    Now retrieving city # 515
    Now retrieving city # 516
    Now retrieving city # 517
    Now retrieving city # 518
    Now retrieving city # 519
    Now retrieving city # 520
    Now retrieving city # 521
    Now retrieving city # 522
    Now retrieving city # 523
    Now retrieving city # 524
    Now retrieving city # 525
    Now retrieving city # 526
    Now retrieving city # 527
    Now retrieving city # 528
    Now retrieving city # 529
    Now retrieving city # 530
    Now retrieving city # 531
    Now retrieving city # 532
    Now retrieving city # 533
    Now retrieving city # 534
    Now retrieving city # 535
    Now retrieving city # 536
    Now retrieving city # 537
    Now retrieving city # 538
    Now retrieving city # 539
    Now retrieving city # 540
    Now retrieving city # 541
    Now retrieving city # 542
    Now retrieving city # 543
    Now retrieving city # 544
    Now retrieving city # 545
    Now retrieving city # 546
    Now retrieving city # 547
    Now retrieving city # 548
    Now retrieving city # 549



```python
new_city1['City Name'].astype(str).values.tolist()

```




    ['Lavrentiya',
     'Zhigansk',
     'Porto Novo',
     'Ushuaia',
     'Busselton',
     'Hilo',
     'Yellowknife',
     'Georgetown',
     'Batasan',
     'Tuktoyaktuk',
     'Urdzhar',
     'Bredasdorp',
     'Hermanus',
     'Albany',
     'Edeia',
     'Acarau',
     'Lagoa',
     'Vaini',
     'Quatre Cocos',
     'Qaanaaq',
     'Port-Cartier',
     'Albany',
     'Rikitea',
     'Avarua',
     'Saint George',
     'Jamestown',
     'Cabo San Lucas',
     'Hanyang',
     'San Patricio',
     'Katsuura',
     'San Borja',
     'Busselton',
     'Fort Frances',
     'Ushuaia',
     'Yaan',
     'New Norfolk',
     'Boguchany',
     'Vaini',
     'Busselton',
     'Ushuaia',
     'Albany',
     'Tsihombe',
     'Kelowna',
     'Hukuntsi',
     'Georgetown',
     'Ushuaia',
     'Kapaa',
     'Ulaanbaatar',
     'Albany',
     'Athy',
     'Rikitea',
     'Vao',
     'Esfahan',
     'Fortuna',
     'Minab',
     'Mar Del Plata',
     'Bluff',
     'Waiouru',
     'Vaini',
     'Albany',
     'Matara',
     'Iqaluit',
     'Kachiry',
     'Kaliua',
     'Hobart',
     'Katsuura',
     'Nyurba',
     'Hermanus',
     'Taby',
     'Cidreira',
     'Punta Arenas',
     'Mataura',
     'Inhambane',
     'Anito',
     'Port Elizabeth',
     'Saskylakh',
     'Bafoulabe',
     'Matamoros',
     'Khatanga',
     'Rikitea',
     'Avarua',
     'Dongning',
     'Ambodifototra',
     'Taolanaro',
     'Torbay',
     'Omboue',
     'Pisco',
     'Ushuaia',
     'Cherskiy',
     'Kodiak',
     'Thompson',
     'Busselton',
     'Tsihombe',
     'Mys Shmidta',
     'Krasnoselkup',
     'Nouadhibou',
     'Bengkulu',
     'Butaritari',
     'La Ronge',
     'Ushuaia',
     'Hedaru',
     'Hilo',
     'Roald',
     'Port Hardy',
     'Batagay',
     'Miri',
     'Wanning',
     'Srednekolymsk',
     'Cockburn Town',
     'Cap Malheureux',
     'Ushuaia',
     'Yulara',
     'Osa',
     'Sao Filipe',
     'Nenjiang',
     'Peruibe',
     'Yellowknife',
     'Uvat',
     'Mataura',
     'Grand River South East',
     'Bluff',
     'East London',
     'Outjo',
     'Port Blair',
     'Yerbogachen',
     'Tasiilaq',
     'Kysyl-Syr',
     'Samusu',
     'Pevek',
     'Mayskiy',
     'Green Valley',
     'Kaitangata',
     'Higuey',
     'Ribeira Grande',
     'Taolanaro',
     'Mar Del Plata',
     'Albany',
     'Narsaq',
     'Kapaa',
     'Moerai',
     'Komsomolskiy',
     'Hobart',
     'Ponta Delgada',
     'Bluff',
     'East London',
     'Cape Town',
     'Nemuro',
     'Yulara',
     'Gushikawa',
     'Bambous Virieux',
     'Kapaa',
     'Ushuaia',
     'Cockburn Town',
     'Santana',
     'Punta Arenas',
     'Chuy',
     'Jamestown',
     'Temaraia',
     'Mataura',
     'Talnakh',
     'Diapaga',
     'Alice Springs',
     'Batouri',
     'Ruatoria',
     'Alyangula',
     'La Ronge',
     'A',
     'Rikitea',
     'Kawalu',
     'Punta Arenas',
     'Arlit',
     'Tiksi',
     'Qaanaaq',
     'Clyde River',
     'Labuhan',
     'Kapaa',
     'Jamestown',
     'Taolanaro',
     'Chagda',
     'Buala',
     'Esperance',
     'Henties Bay',
     'Kahului',
     'Klaksvik',
     'Vypolzovo',
     'Suamico',
     'Ningan',
     'Hilo',
     'Attawapiskat',
     'Ushuaia',
     'Cape Town',
     'Busselton',
     'Barentsburg',
     'Rikitea',
     'Rikitea',
     'Ostrovnoy',
     'Saint-Louis',
     'Jamestown',
     'Rikitea',
     'Cape Town',
     'Ushuaia',
     'Pevek',
     'Laguna',
     'Port Alfred',
     'Ajaigarh',
     'Los Llanos De Aridane',
     'Cape Town',
     'Mar Del Plata',
     'Pevek',
     'Belushya Guba',
     'Puerto Cabezas',
     'Butaritari',
     'New Norfolk',
     'Zheleznovodsk',
     'Yarkovo',
     'Veletma',
     'Kahului',
     'Busselton',
     'Busselton',
     'Narsaq',
     'Zhezkazgan',
     'Kapaa',
     'Port Lincoln',
     'Punta Arenas',
     'Sumenep',
     'Busselton',
     'Port Alfred',
     'Puerto Ayora',
     'Moyo',
     'Cape Town',
     'Rikitea',
     'Namibe',
     'Ushuaia',
     'Illoqqortoormiut',
     'Hilo',
     'Aksu',
     'Elko',
     'Atuona',
     'Barrow',
     'Lagoa',
     'Faanui',
     'Jamestown',
     'Codrington',
     'Taolanaro',
     'Tuktoyaktuk',
     'Vardo',
     'Hobart',
     'Taolanaro',
     'Rikitea',
     'Hermanus',
     'Uvira',
     'Warqla',
     'Mergui',
     'Constitucion',
     'Belushya Guba',
     'Puerto Ayora',
     'San Cristobal',
     'Rikitea',
     'Bintulu',
     'Talnakh',
     'Tarko-Sale',
     'Vaini',
     'Rikitea',
     'Palabuhanratu',
     'Rungata',
     'Grand River South East',
     'Cockburn Town',
     'Longyearbyen',
     'Ushuaia',
     'Faanui',
     'Belushya Guba',
     'Albany',
     'Dunedin',
     'Clyde River',
     'Chemax',
     'Hermanus',
     'Kaitangata',
     'Bluff',
     'Punta Arenas',
     'Upernavik',
     'Butaritari',
     'Sorland',
     'Torbay',
     'Carnarvon',
     'Rikitea',
     'Saint-Augustin',
     'Jurm',
     'Cape Town',
     'Illoqqortoormiut',
     'Aykhal',
     'Vaini',
     'Butaritari',
     'Menongue',
     'Georgetown',
     'Bredasdorp',
     'Victoria',
     'Mataura',
     'Riaba',
     'Lebu',
     'Busselton',
     'Tsihombe',
     'Bitung',
     'Presidencia Roque Saenz Pena',
     'Poum',
     'Neuquen',
     'Rikitea',
     'Busselton',
     'Bousse',
     'Soyo',
     'Busselton',
     'Mataura',
     'Ponta Do Sol',
     'Faya',
     'Cape Town',
     'Albany',
     'Celestun',
     'Saint-Philippe',
     'Sisimiut',
     'Punta Arenas',
     'Kloulklubed',
     'Rikitea',
     'Puerto Ayora',
     'Butaritari',
     'Iqaluit',
     'Mitu',
     'Rikitea',
     'Saint-Pierre',
     'Rikitea',
     'Vaini',
     'North Bend',
     'Pemangkat',
     'Punta Arenas',
     'Kodiak',
     'Kholmogory',
     'Husavik',
     'Ushuaia',
     'Bargal',
     'Namibe',
     'Hermanus',
     'Rikitea',
     'Qaanaaq',
     'Dingle',
     'Jamestown',
     'Contai',
     'Lebu',
     'Punta Arenas',
     'Busselton',
     'Ormara',
     'Tiksi',
     'Ambovombe',
     'Blue Springs',
     'Broome',
     'Yulara',
     'Bluff',
     'Fevralsk',
     'Georgetown',
     'Taolanaro',
     'Tirano',
     'Atuona',
     'Hobart',
     'Vila Velha',
     'Ushuaia',
     'Saleaula',
     'Bonthe',
     'Jutai',
     'Nadym',
     'Ziway',
     'Vardo',
     'Ushuaia',
     'Bluff',
     'Chenghai',
     'Victoria',
     'Butaritari',
     'Butaritari',
     'Buala',
     'Busselton',
     'Faanui',
     'Zaysan',
     'Leningradskiy',
     'Dikson',
     'Fuling',
     'Rikitea',
     'Clyde River',
     'Ushuaia',
     'Port Alfred',
     'Busselton',
     'Hofn',
     'Pandelys',
     'Cherskiy',
     'Avarua',
     'Bluff',
     'Vaini',
     'Hilo',
     'Jamestown',
     'Rikitea',
     'Rikitea',
     'Maceio',
     'Castro',
     'Punta Arenas',
     'Faanui',
     'Mataura',
     'Yuryevets',
     'Sentyabrskiy',
     'Port Hedland',
     'Rikitea',
     'Jalu',
     'Atuona',
     'Rikitea',
     'Reconquista',
     'Zhanaozen',
     'Ushuaia',
     'Punta Arenas',
     'Akdepe',
     'Rikitea',
     'Vaini',
     'Aryanah',
     'Ahipara',
     'Thompson',
     'Hilo',
     'Cabo San Lucas',
     'Saint-Philippe',
     'Eydhafushi',
     'Stokmarknes',
     'Lufilufi',
     'Mezhdurechensk',
     'The Pas',
     'San Quintin',
     'Borama',
     'Hermanus',
     'Kjollefjord',
     'Bambous Virieux',
     'Punta Arenas',
     'Mani',
     'Rikitea',
     'Ribeira Grande',
     'Barrow',
     'Ushuaia',
     'Lang Son',
     'Teya',
     'Port Alfred',
     'Yumen',
     'Mlandizi',
     'Sur',
     'Ixtapa',
     'Garm',
     'Hermanus',
     'Rikitea',
     'Kaitangata',
     'Korla',
     'Ushuaia',
     'Rawson',
     'Georgetown',
     'Mumford',
     'San Ramon',
     'Butaritari',
     'Castro',
     'Dikson',
     'Alofi',
     'Ushuaia',
     'Meyungs',
     'Hovd',
     'Khatanga',
     'Lebu',
     'Victoria',
     'Taolanaro',
     'Vaini',
     'Cape Town',
     'Sampit',
     'Port Alfred',
     'Souillac',
     'Rikitea',
     'Tiksi',
     'Haines Junction',
     'Punta Arenas',
     'Barentsburg',
     'Chokurdakh',
     'Udpura',
     'Kodiak',
     'Kimberley',
     'Taolanaro',
     'Hobart',
     'Ribeira Grande',
     'Constitucion',
     'Saskylakh',
     'Hobart',
     'Punta Arenas',
     'Hilo',
     'Luderitz',
     'Hithadhoo',
     'Salalah',
     'Baruun-Urt',
     'Vardo',
     'Mount Isa',
     'Los Llanos De Aridane',
     'Albany',
     'Rikitea',
     'Necochea',
     'Elk',
     'Amderma',
     'Rungata',
     'Prince Rupert',
     'Mar Del Plata',
     'Atuona',
     'Pousat',
     'Meulaboh',
     'Albany',
     'Bredasdorp',
     'Sinnamary',
     'Taolanaro',
     'Rikitea',
     'Hazorasp',
     'Bluff',
     'Atuona',
     'Hobart',
     'Rikitea',
     'Rikitea',
     'Attawapiskat',
     'Wajir',
     'Thompson',
     'Mount Gambier',
     'Rikitea',
     'Jamestown',
     'Saint-Georges',
     'Kalmunai',
     'Nuuk',
     'Savannah Bight',
     'Albany',
     'Taolanaro',
     'Mandal',
     'Sao Filipe',
     'Canto Do Buriti',
     'Hilo',
     'Bambous Virieux',
     'La Ronge',
     'Moerai',
     'Sheridan',
     'Carnarvon',
     'East London',
     'Faanui',
     'Hermanus',
     'New Norfolk',
     'Gigmoto',
     'Lingyuan',
     'Ushuaia',
     'Cape Town',
     'Albany',
     'Santarem',
     'Coihaique',
     'Hermanus',
     'Koyilandi']




```python
url = "http://api.openweathermap.org/data/2.5/weather?"
units = "Imperial"

# Build partial query URL
query_url = url + "units=" + units+"&APPID=" + wkey+"&q="
query_url
```




    'http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=d3954bc3f5f2bc6f9f356bfd8aee0020&q='




```python
weather_data = []
cities=[new_city1['City Name']]

for city in cities:
    response = req.get(query_url + city).json()
    weather_data.append(response)
    

weather_data
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-11-9ae255811027> in <module>()
          3 
          4 for city in cities:
    ----> 5     response = req.get(query_url + city).json()
          6     weather_data.append(response)
          7 


    NameError: name 'req' is not defined

