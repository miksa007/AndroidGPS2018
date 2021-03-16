# AndroidGPS2018

Tutoriaali paikannuksen käyttöönottoon tarpeellisilla oikeuksilla. Tämä ohje rakentaa softan, jolla nappulaa painamalla saadaan gps-koordinaatit laitteen ruutuun.

Paikannus tarvii oikeuden `AndroidManifest.xml` tiedostoon


    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>

Seuraavaksi muutoksia MainActivity.java tiedostooon. Aluksi esitellään pari luokkamuuttujaa:

    private LocationManager mLocationManager;
    private LocationListener mLocationListener;
    private Location mLocation;
    public static final int MY_PERMISSIONS_REQUEST_LOCATION = 98;

    private TextView paikkaTextView;
    private Button haePaikkaButton;

    private Context context;

edellä mainitut ovat kolme paikannukseen liittyvää ja yksi oikeuksiin liittyvä.
Edellisiin liittyen määritellään kyseiset muuttujat:

     mLocationManager = (LocationManager) this.getSystemService(Context.LOCATION_SERVICE);

Ja kuuntelijan määrittely alla onkin hieman pidempi, mutta Android Studio generoi tämänkin lähes kokonaan:

    mLocationListener=new LocationListener() {
            @Override
            public void onLocationChanged(Location location) {
                Log.d("lokapaikka", ("paikka on muuttunut"+location.getLatitude()+", "+location.getLongitude()));
                mLocation=location;
            }

            @Override
            public void onStatusChanged(String s, int i, Bundle bundle) {

            }

            @Override
            public void onProviderEnabled(String s) {

            }

            @Override
            public void onProviderDisabled(String s) {

            }
        };

Edellisessä koodinpätkässä on `LocationChanged(Location location)` -metodia kutsutaan aina kun sijainti muuttuu. Jos sovelluksen halutaan tekevän jotain tällöin niin tuohon metodiin voi tehdä toimintaan liittyvän metodikutsun. Tässä esimerkissä location tallennetaan luokkamuuttujaan myöhempää käyttöä varten.

Seuraavaksi siirrytään oikeuksien hallintaan, jotta saadaan paikannukselle oikeudet käyttäjältä. Oikeudet ja niiden olemassaolo pitäisi tarkistaa aina käytön yhteydessä. Seuraava metodi kysyLupaa() on rakennettu dokumentaation perusteella omaksi metodiksi. Näin sitä voidaan hyväksikäyttää oikeuksien tarkistamiseen aina tarpeen mukaan.

        public boolean kysyLupaa(final Context context){
        Log.d("lokasofta", "kysyLupaa()");
        // Here, thisActivity is the current activity
        if (ContextCompat.checkSelfPermission(this,
                Manifest.permission.ACCESS_FINE_LOCATION)
                != PackageManager.PERMISSION_GRANTED) {

            Log.d("lokasofta", " Permission is not granted");
            // Should we show an explanation?
            if (ActivityCompat.shouldShowRequestPermissionRationale(this,
                    Manifest.permission.ACCESS_FINE_LOCATION)) {

                // Show an explanation to the user *asynchronously* -- don't block
                // this thread waiting for the user's response! After the user
                // sees the explanation, try again to request the permission.
                Log.d("lokasofta", "Kerran kysytty, mutta ei lupaa... Nyt ei kysytä uudestaan");

            } else {
                Log.d("lokasofta", " Request the permission");
                // No explanation needed; request the permission
                ActivityCompat.requestPermissions(this,
                        new String[]{Manifest.permission.ACCESS_FINE_LOCATION},
                        MY_PERMISSIONS_REQUEST_LOCATION);

                // MY_PERMISSIONS_REQUEST_READ_LOCATION is an
                // app-defined int constant. The callback method gets the
                // result of the request.

            }
            return false;
        } else {

            Log.d("lokasofta", "Permission has already been granted");
            return true;
        }

    }

Edellä mainitussa kysytään vain `ACCESS_FINE_LOCATION` -oikeutta, mutta muiden oikeuksien tarkistus tapahtuu saman kaavan mukaan.

Alkaen Android versiosta 6.0 sovellus kysyy aina käytönyhteydessä käyttäjältä oikeutta "vaarallisen" toiminnallisuuden käyttöön. Tämä kysely on rakennettu seuraavaan yläluokasta periytettävään metodiin, jota kysellään automaattisesti tarpeen mukaan kysyLupaa() -metodista:

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        Log.d("lokasofta ", "onRequestPermissionsResult()");

        switch (requestCode) {
            case MY_PERMISSIONS_REQUEST_LOCATION: {
                // If request is cancelled, the result arrays are empty.
                if (grantResults.length > 0
                        && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    Log.d("lokasofta", "lupa tuli!");
                    // permission was granted, yay! Do the
                    // location-related task you need to do.
                    if (ContextCompat.checkSelfPermission(this,
                            Manifest.permission.ACCESS_FINE_LOCATION)
                            == PackageManager.PERMISSION_GRANTED) {
                        Log.d("lokasofta", "Haetaan paikkaa tietyin väliajoin");
                        //Request location updates:
                        //mLocationManager.requestLocationUpdates(LocationManager.NETWORK_PROVIDER,0,0,mLocationListener);
                    }
                } else {
                    // permission denied, boo! Disable the
                    // functionality that depends on this permission.
                    Log.d("lokasofta", "Ei tullu lupaa!");
                }
                return;
            }

        }
    }

Nyt tarvittavat paikannukset ja sen käyttöön tarvittavat oikudet pitäisi olla kunnosssa, Eli voidaan aloittaa paikannustiedon käyttö. 
Käyttöliittymä `activity_main.xml` sisältää textview ja Button -komponentit.TODO - Kuva tässä olisi hieno...

    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="24dp"
        android:text="Hello World!"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="24dp"
        android:text="Hae paikka"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/textView" />

`MainActivity.java` tiedostossa liitetään komponentit TextViev ja Button toimintaan mukaan:

    helloTextView=findViewById(R.id.textView);
    haePaikkaButton=findViewById(R.id.button);

Lisäksi tehdään buttonille kuuntelija:

    context=this; 
    haePaikkaButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                //tarkistetaan lupa
                try {
                    kysyLupaa(context);
                    //Tässä kaksi eri tapaa paikannukseen
                    mLocationManager.requestLocationUpdates(LocationManager.NETWORK_PROVIDER, 0, 0, mLocationListener);
                    //mLocationManager.requestLocationUpdates(LocationManager.GPS_PROVIDER, 0, 0, mLocationListener);
                    if(mLocation!=null) {
                        helloTextView.setText(mLocation.getLatitude() + ", " + mLocation.getLongitude());
                    }else{
                        helloTextView.setText("Paikka ei vielä saatavilla... Kokeile uudestaan");
                    }
                }catch (SecurityException e){
                    Log.d("lokasofta", "Virhe: Sovelluksella ei ollut oikeuksia lokaatioon");
                }
            }
        });

Koodissa `mLocationManager.requestLocationUpdates(LocationManager.GPS_PROVIDER, 0, 0, mLocationListener);` ottaa käyttöön paikannuksen ja määrittelee mm. miten usein paikkaa kysellään. Paikkatiedon `mLocation` saaminen kestää vähän aikaa, joten if-lause estää sovelluksen virheet. Try - catch -rakenne on kuitenkin pakollinen kun paikannuspalvelua käytetään. Mutta jos kaikki menee oikein niin sovellus kirjoittaa nappulan painalluksella GPS-koordinaatit ruutuun. Jos paikkaa ei löydy, niin vaihtoehtoisesti voi käyttää ` mLocationManager.requestLocationUpdates(LocationManager.NETWORK_PROVIDER, 0, 0, mLocationListener);´


Kommentteja?
(Toimi 15_3_2021)