<script>
    //Grap data
    import {onMount} from 'svelte';
    import {fetchMovies} from '../api';
    //config
    import { IMAGE_BASE_URL, BACKDROP_SIZE, POSTER_SIZE} from '../config';
    //components
    import Hero from '../components/Hero.svelte';
    import Search from '../components/Search.svelte';
    import Grid from '../components/Grid.svelte';
    import Thumb from '../components/Thumb.svelte';
    import LoadMoreButton from '../components/LoadMoreButton.svelte';
    import Spinner from '../components/Spinner.svelte';

    let movies = {movies: []};
    let isLoading;
    let searchTerm = "";
    let error;
   
    //FETCH THE MOVIES (TOTAL, CURRENT, ...)
    const handleFetchMovies = async (loadMore, searchTerm) => {
        try {
            isLoading = true;
            error = false;
            movies = await fetchMovies(movies, loadMore, searchTerm);
          // console.log(movies);
        } catch (err) {
            error = true;
            console.log(err);
        }
        isLoading = false;
    };

    //fonction de recherche
    const handleSearch = event => {
        searchTerm = event.detail.searchText; //récupérer object.detail from dispatch event
        movies.movies = [];//
        handleFetchMovies(false, searchTerm);
    }

    //CHARGER PLUS
    const handleLoadMore = () => handleFetchMovies(true, searchTerm);

    onMount(async () => {
        const sessionMovies = window.sessionStorage.getItem('svelte-movies');
        if(sessionMovies){
            console.log('grab from session');
            movies = JSON.parse(sessionMovies);
        }else{
            console.log('grab from API');
            handleFetchMovies(false, searchTerm);
        }
      

       // movies = await fetchMovies(movies, false, searchTerm);
    });

    $: {
        if(movies.movies.length > 0){
            window.sessionStorage.setItem('svelte-movies', JSON.stringify(movies));
        }
    }

</script>
    <!-- show hero-->
    {#if error}
            <p>sometihing went wrong</p>
        {:else}
            {#if movies.heroImage && !searchTerm }
            <Hero 
                image={`${IMAGE_BASE_URL}${BACKDROP_SIZE}${movies.heroImage.backdrop_path}`}
                title={movies.heroImage.original_title}
                text={movies.heroImage.overview} />
            {/if}
    {/if}

    

    <Search on:search={handleSearch} /> 


    <Grid header={searchTerm ? 'search Result' : 'Popular Movies'}>
        {#each movies.movies as movie}
        <Thumb
            clickable
            image={movie.poster_path && IMAGE_BASE_URL + POSTER_SIZE + movie.poster_path}
            movieId = {movie.id}/>
        {/each}
    </Grid>

    {#if isLoading}
        <Spinner />
    {/if}

    {#if !isLoading && movies.currentPage < movies.totalPages}
        <LoadMoreButton on:loadMore={handleLoadMore}>
            Load More
        </LoadMoreButton>
    {/if}
    

   
<style>

</style>
