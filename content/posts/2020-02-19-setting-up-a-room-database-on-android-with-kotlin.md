---
title: Setting up a Room database on Android with Kotlin
author: Jesús Sánchez
type: post
date: 2020-02-19T14:13:46+00:00
url: /setting-up-a-room-database-on-android-with-kotlin/
categories:
  - Android
tags:
  - Android
  - Kotlin
  - Room

---
Setup the required dependencies. You&#8217;ll need the room-runtime and room-compiler.

## Dependencies & Gradle plugin

{{< highlight kotlin "lineos=true" >}}
// app/build.gradle
apply plugin: 'kotlin-kapt'

dependencies {
    def room_version = "2.2.3"
    implementation "androidx.room:room-runtime:$room_version"
    kapt "androidx.room:room-compiler:$room_version"
}
{{< / highlight >}}

## Creating the database

{{< highlight kotlin "lineos=true" >}}
// Book.kt
@Entity
data class Book(@PrimaryKey val id: UUID = UUID.randomUUID(),
                var title: String = "",
                var date: Date = Date(),
                var isComplete: Boolean = false)
{{< / highlight >}}

{{< highlight kotlin "lineos=true" >}}
// database/BookDatabase.kt
@Database(entities = [ Book::class ], version=1)
@TypeConverters(BookTypeConverters::class)
abstract class BookDatabase : RoomDatabase() {
    abstract fun bookDao(): BookDao
}
{{< / highlight >}}

## Type Converters

{{< highlight kotlin "lineos=true" >}}
// database/BookTypeConverters.kt
class BookTypeConverters {
    @TypeConverter
    fun fromDate(date: Date?): Long? {
        return date?.time
    }

    @TypeConverter
    fun toDate(millisSinceEpoch: Long?): Date? {
        return millisSinceEpoch?.let {
            Date(it)
        }
    }

    @TypeConverter
    fun fromUUID(uuid: UUID?): String? {
        return uuid?.toString()
    }

    @TypeConverter
    fun toUUID(uuid: String?): UUID? {
        return UUID.fromString(uuid)
    }
}
{{< / highlight >}}

## DAO (Data Access Object)

{{< highlight kotlin "lineos=true" >}}
// database/BookDao.kt
@Dao
interface BookDao {
    @Query("SELECT * FROM book")
    fun getBooks(): LiveData&lt;List&lt;Book>>

    @Query("SELECT * FROM book WHERE id=(:id)")
    fun getBook(id: UUID): LiveData&lt;Book?>
}
{{< / highlight >}}

## Repository

{{< highlight kotlin "lineos=true" >}}
// BookRepository.kt
private const val DATABASE_NAME = "book-database"

class BookRepository private constructor(context: Context) {

    private val database : BookDatabase = Room.databaseBuilder(
        context.applicationContext,
        BookDatabase::class.java,
        DATABASE_NAME
    ).build()

    private val bookDao = database.bookDao()

    fun getBooks(): LiveData&lt;List&lt;Book>> = bookDao.getBooks()

    fun getBook(id: UUID): LiveData&lt;Book?> = bookDao.getBook(id)

    companion object {
        private var INSTANCE: BookRepository? = null

        fun initialize(context: Context) {
            if (INSTANCE == null) {
                INSTANCE = BookRepository(context)
            }
        }

        fun get(): BookRepository {
            return INSTANCE ?: throw IllegalStateException("BookRepository must be initialized")
        }
    }
}
{{< / highlight >}}

## Application Subclass

{{< highlight kotlin "lineos=true" >}}
// BookshelfApplication.kt
class BookshelfApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        BookRepository.initialize(this)
    }
}
{{< / highlight >}}

manifests/AndroidManifest.xml

{{< highlight xml "lineos=true" >}}
<application
    android:name=".BookshelfApplication"
    android:allowBackup="true">
</application>
{{< / highlight >}}

## ViewModel

{{< highlight kotlin "lineos=true" >}}
// BookListViewModel.kt
class BookListViewModel : ViewModel() {
    private val bookRepository = BookRepository.get()
    val bookListLiveData = bookRepository.getBooks()
}
{{< / highlight >}}

## List Fragment

{{< highlight kotlin "lineos=true" >}}
// BookListFragment.kt
private const val TAG = "BookListFragment"

class BookListFragment : Fragment() {

    private lateinit var bookRecyclerView: RecyclerView
    private var adapter: BookAdapter? = BookAdapter(emptyList())

    private val bookListViewModel: BookListViewModel by lazy {
        ViewModelProvider(this).get(BookListViewModel::class.java)
    }

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        val view = inflater.inflate(R.layout.fragment_book_list, container, false)
        bookRecyclerView = view.findViewById(R.id.book_recycler_view) as RecyclerView
        bookRecyclerView.layoutManager = LinearLayoutManager(context)
        bookRecyclerView.adapter = adapter

        return view
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        bookListViewModel.bookListLiveData.observe(
            viewLifecycleOwner,
            Observer { books ->
                books?.let {
                    Log.i(TAG, "Got books ${books.size}")
                    updateUI(books)
                }
            }
        )
    }

    private fun updateUI(books: List&lt;Book>) {
        adapter = BookAdapter(books)
        bookRecyclerView.adapter = adapter
    }

    private inner class BookHolder(view: View)
        : RecyclerView.ViewHolder(view), View.OnClickListener {

        private lateinit var book: Book

        private val titleTextView = itemView.findViewById&lt;TextView>(R.id.book_title)
        private val dateTextView = itemView.findViewById&lt;TextView>(R.id.book_date)
        private val completedImageView = itemView.findViewById&lt;ImageView>(R.id.book_completed)

        init {
            itemView.setOnClickListener(this)
        }

        fun bind(book: Book) {
            this.book = book
            titleTextView.text = this.book.title

            dateTextView.text = android.text.format.DateFormat.format("EEEE, MMM d, yyyy", this.book.date)

            if (book.isCompleted) {
                completedImageView.visibility = View.VISIBLE
            } else {
                completedImageView.visibility = View.GONE
            }
        }

        override fun onClick(v: View?) {
            Toast.makeText(context, "${book.title} pressed", Toast.LENGTH_SHORT).show()
        }

    }

    private inner class BookAdapter(var books: List&lt;Book>) : RecyclerView.Adapter&lt;BookHolder>() {

        override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): BookHolder {
            val view = layoutInflater.inflate(R.layout.list_item_book, parent, false)
            return BookHolder(view)
        }

        override fun getItemCount() = books.size

        override fun onBindViewHolder(holder: BookHolder, position: Int) {
            val book = books[position]
            holder.bind(book)
        }

    }

    companion object {
        fun newInstance() : BookListFragment {
            return BookListFragment()
        }
    }
}
{{< / highlight >}}
