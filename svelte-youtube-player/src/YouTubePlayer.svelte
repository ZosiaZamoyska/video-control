<script>
    import { onMount } from 'svelte';
    let player;
    let videoId = 'kzwwmMPRjbk';
    let segments = [
        { start: 0, end: 35, label: 'explanation' },
        { start: 35, end: 48, label: 'slip knot' },
        { start: 48, end: 51, label: 'chain (1)' },
        { start: 51, end: 52, label: 'chain (2)' },
        { start: 52, end: 53, label: 'chain (3)' },
        { start: 54, end: 55, label: 'chain (4)' },
        { start: 56, end: 58, label: 'chain (5)' }
    ];
    let currentSegmentIndex = 0;
    let isPausedManually = false;
    let segmentInterval;

    function onYouTubeIframeAPIReady() {
        player = new YT.Player('player', {
            height: '360',
            width: '640',
            videoId,
            events: {
                onReady: onPlayerReady,
                onStateChange: onPlayerStateChange
            }
        });
    }

    function onPlayerReady(event) {
        console.log('Player is ready');
        playSegment(currentSegmentIndex);
    }

    function onPlayerStateChange(event) {
        if (event.data === YT.PlayerState.PLAYING) {
            checkSegmentEnd();
        }
    }

    function checkSegmentEnd() {
        clearInterval(segmentInterval);
        segmentInterval = setInterval(() => {
            let segment = segments[currentSegmentIndex];
            if (!isPausedManually && player.getCurrentTime() >= segment.end) {
                pauseVideo();
                clearInterval(segmentInterval);
            }
        }, 500);
    }

    function playSegment(index) {
        if (index < segments.length) {
            currentSegmentIndex = index;
            let segment = segments[index];
            isPausedManually = false; // Reset pause state when playing new segment
            player.seekTo(segment.start, true);
            player.playVideo();
            checkSegmentEnd();
        }
    }

    function prevSegment() {
        if (currentSegmentIndex > 0) {
            playSegment(currentSegmentIndex - 1);
        }
    }

    function nextSegment() {
        if (currentSegmentIndex < segments.length - 1) {
            playSegment(currentSegmentIndex + 1);
        }
    }

    function pauseVideo() {
        isPausedManually = true;
        player.pauseVideo();
    }

    function playVideo() {
        isPausedManually = false;
        let currentTime = player.getCurrentTime();
        let segment = segments[currentSegmentIndex];
        
        if (currentTime >= segment.end && currentSegmentIndex < segments.length - 1) {
            playSegment(currentSegmentIndex + 1);
        } else {
            player.playVideo();
        }
    }

    function rewindVideo(seconds = 1) {
        let currentTime = player.getCurrentTime();
        player.seekTo(currentTime - seconds, true);
    }

    onMount(() => {
        if (!window.YT) {
            let tag = document.createElement('script');
            tag.src = 'https://www.youtube.com/iframe_api';
            let firstScriptTag = document.getElementsByTagName('script')[0];
            firstScriptTag.parentNode.insertBefore(tag, firstScriptTag);
            window.onYouTubeIframeAPIReady = onYouTubeIframeAPIReady;
        } else {
            onYouTubeIframeAPIReady();
        }
    });
</script>

<div>
    <div id="player"></div>
    <button on:click={pauseVideo}>Pause</button>
    <button on:click={playVideo}>Play</button>
    <button on:click={() => rewindVideo(1)}>Undo (Rewind 1s)</button>
    <button on:click={prevSegment}>Prev Step</button>
    <button on:click={nextSegment}>Next Step</button>
    <p>Current Step: {segments[currentSegmentIndex].label}</p>
</div>
