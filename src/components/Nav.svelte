<script>
	export let segment;
</script>

<header class='
    bg-gray-300
    fixed
    flex
    items-stretch
    justify-between
    left-0
    max-w-6xl
    mt-6
    mx-auto
    px-12
    py-4
    right-0
    top-0
'>
    <p class='
        font-bold
        font-body
        text-4xl
        text-gray-900
        tracking-wider
    '>
        <a href='.'>MT</a>
    </p>
    <nav class='
        flex
        items-stretch
    '>
        <ul class='
            font-display
            flex
            items-stretch
            text-2xl
            text-gray-900
        '>
            <li class='
                flex
                items-stretch
                mr-8
            '>
                <a class='
                    flex
                    items-center
                    
                    {segment === "work" ? "underline" : ""}
                ' href='work'>work</a>
            </li>
            <li class='
                flex
                items-stretch
            '>
                <a rel=prefetch class='
                    flex
                    items-center
                    
                    {segment === "blog" ? "underline" : ""}
                ' href='blog'>blog</a>
            </li>
        </ul>
    </nav>
</header>