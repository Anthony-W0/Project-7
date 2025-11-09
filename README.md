MainActivity.kt 
package com.example.lab_6

import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.bumptech.glide.Glide
import com.loopj.android.http.AsyncHttpClient
import com.loopj.android.http.JsonHttpResponseHandler
import cz.msebera.android.httpclient.Header
import org.json.JSONObject
import kotlin.random.Random
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.ImageView
import android.widget.TextView


// Represents a single Pokemon object with its name, types, and image URL
data class Pokemon(
    val name: String,
    val types: List<String>,
    val imageUrl: String
)

// Adapter for the RecyclerView that displays the list of Pokemon
class PokemonAdapter(private val pokemonList: MutableList<Pokemon>) :
    RecyclerView.Adapter<PokemonAdapter.PokemonViewHolder>() {

    class PokemonViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val nameTextView: TextView = itemView.findViewById(R.id.nameText)
        val typeTextView: TextView = itemView.findViewById(R.id.typeText)
        val imageView: ImageView = itemView.findViewById(R.id.Pokeview)
    }


    //Loads one Pokemon layout (pokemon_item.xml) into the RecyclerView
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): PokemonViewHolder {
        val itemView = LayoutInflater.from(parent.context)
            .inflate(R.layout.pokemon_item, parent, false)
        return PokemonViewHolder(itemView)
    }

    // Take one Pokemon from the list and fills in the layout
    //name
    //type
    //image
    //Glide downloads and displays the image from the URL
    override fun onBindViewHolder(holder: PokemonViewHolder, position: Int) {
        val currentPokemon = pokemonList[position]
        holder.nameTextView.text = currentPokemon.name
        holder.typeTextView.text = currentPokemon.types.joinToString(", ")
        Glide.with(holder.itemView)
            .load(currentPokemon.imageUrl)
            .centerInside()
            .into(holder.imageView)
    }

    //Tells the RecyclerView how many items to display
    override fun getItemCount() = pokemonList.size
}

class MainActivity : AppCompatActivity() {

    private val baseUrl = "https://pokeapi.co/api/v2/pokemon/"
    private lateinit var pokemonRecyclerView: RecyclerView
    //UI element that displays the scrolling feed of Pokemon

    private lateinit var pokemonAdapter: PokemonAdapter
    // the bridge between data and the RecyclerView
    private val client = AsyncHttpClient()
    //sends and receives HTTP requests

    private val pokemonList = mutableListOf<Pokemon>()
    //Stores all Pokemon objects that have been fetched

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        pokemonRecyclerView = findViewById(R.id.pokemonRecyclerView)
        pokemonAdapter = PokemonAdapter(pokemonList)
        pokemonRecyclerView.layoutManager = LinearLayoutManager(this)
        pokemonRecyclerView.adapter = pokemonAdapter

        // Load the first Pokémon
        fetchRandomPokemon()

        // Add scroll listener — fetch new Pokémon when reaching bottom
        pokemonRecyclerView.addOnScrollListener(object : RecyclerView.OnScrollListener() {
            override fun onScrolled(recyclerView: RecyclerView, dx: Int, dy: Int) {
                super.onScrolled(recyclerView, dx, dy)
                if (!recyclerView.canScrollVertically(1)) {
                    // When you scroll to the bottom, load another random Pokémon
                    fetchRandomPokemon()
                }
            }
        })
    }

        // Fetch a random Pokémon from the API and add it to the list
    private fun fetchRandomPokemon() {
        val id = Random.nextInt(1, 899)
        val url = "$baseUrl$id/"

            // Sends an HTTP GET request to the specified URL and handles the response
        client.get(url, object : JsonHttpResponseHandler() {
            override fun onSuccess(statusCode: Int, headers: Array<out Header>?, response: JSONObject) {
                val name = response.optString("name", "unknown")
                    .replaceFirstChar { it.uppercaseChar() }
                val sprites = response.optJSONObject("sprites")
                val spriteUrl = sprites?.optString("front_default") ?: ""
                val typesArray = response.optJSONArray("types")
                val typesList = mutableListOf<String>()
                if (typesArray != null) {
                    for (i in 0 until typesArray.length()) {
                        val typeObj = typesArray.optJSONObject(i)?.optJSONObject("type")
                        val typeName = typeObj?.optString("name")
                        if (!typeName.isNullOrEmpty()) {
                            typesList.add(typeName.replaceFirstChar { it.uppercaseChar() })
                        }
                    }
                }

                val pokemon = Pokemon(name, typesList, spriteUrl)
                pokemonList.add(pokemon)
                pokemonAdapter.notifyItemInserted(pokemonList.size - 1)
            }

            override fun onFailure(statusCode: Int, headers: Array<out Header>?, throwable: Throwable?, errorResponse: JSONObject?) {
                println("Request failed: $statusCode")
            }
        })
    }
}
