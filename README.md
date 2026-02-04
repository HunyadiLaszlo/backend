laravel new project_neve

Migrations php

	$table->charset('utf8');
	$table->collation('utf8_hungarian_ci');

	$table->string('nev')->unique();
	
	$table->foreignIdFor(category::class)->nullable()->constrained();

Models

    public function category(): BelongsTo
    {
        return $this->belongsTo(category::class, foreignKey:'az id neve');  //csak akkor kell foreignKey, ha a key nem a tábla nevéből ered pl. categories táblára hivatkozó category_id
    }
	
	    public function products(): HasMany
    {
        return $this->hasMany(product::class, foreignKey:'az id neve');   //csak akkor kell foreignKey, ha a key nem a tábla nevéből ered pl. categories táblára hivatkozó category_id
    }
	
	//Ha nem konvencionális elnevezést használunk, akkor a tábla nevét vagy a primary key-t is itt tudjuk megadni:
	protected $table = 'valami';
	protected primaryKey = 'valami';

	//Dátum esetén
	    protected function casts(): array
    {
        return [
            'szuletesi_datum' => 'date:Y-m-d',
        ];
    }
	
Request

	'name' => ['required' , 'string', 'max:255', 'min:1'],
    'description' => ['sometimes'],
    'stock' => ['sometimes', 'integer', 'min:0',  'max:1000'],
    'price' => ['sometimes', 'integer', 'min:0'],
    'imageUrl' => ['sometimes','unique:products,imageUrl'],
    'category_id' => ['nullable', Rule::exists('categories', 'id')],
	
	public function messages()
    {
        return [
            "price.min" =>  "Negarív ár nem értelmezett",
        ];
    }

	// dátumnál a sometimes mellé kell a nullable is
	'szuletesi_datum' => ['sometimes', 'nullable',  'date'],

Resource

	'id' => $this->id,
    'name' => $this->name,
    'description' => $this->description,
    'stock' => $this->stock,
    'price' => $this->price,
    'imageUrl' => $this->whenNotNull($this->imageUrl),

    'category_id' => $this->category_id,

    'category' => new categoryResource($this->whenLoaded('category')),

	// has many-nél collection-t kell visszaadni!!!!!
	'diak' => studentResource::collection($this->whenLoaded('students')),

	//ha dátum nullable, kell a ?
	'szuletesi_datum' => $this->szuletesi_datum?->format('Y-m-d'),
	
Controller

	//ha a teljes listában is meg kell jeleníteni az altáblát
	
	public function index()
    {
        return productResource::collection(product::with('category')->get());
    }
	
	//egyedlistában az altábla megjelenítése
	
	public function show(product $product)
    {
        return new productResource($product->load('category'));
    }
	
	//ha státuszkódot is vissza kell küldeni
	
	public function destroy(category $category)
    {
        $category->delete();

        return response()->json("sikeres törlés", Response::HTTP_NO_CONTENT);
        
		vagy 
		
		return response()->json(data:"sikeres törlés", status:204);
    }
	
