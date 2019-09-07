<script>
import { onMount } from 'svelte';

let armCircle = '';
let arm = '';
let label = '';
let stopButton = '';
let tracks = '';

let armCircleDimensions = {};
let armCircleCenter = { x: 0, y: 0 };
let armCircleRadius = 0;

let scrollPositionY = 0;
let stopLabelSpin = false;

let allowNeedleGrab = false;

let angles = {
    'One': 30,
    'Two': 33.8,
    'Three': 37.6,
    'Four': 41.4,
    'Five': 45.2
}

onMount(() => {
    arm = document.getElementById('turn');
    armCircle = document.getElementById('armCircle');
    label = document.getElementById('label');
    stopButton = document.getElementById('stopButton');
    
    stopLabelSpin = function() {
        label.setAttribute('class', '');
    }
    
    setTimeout(function() {
        arm.setAttribute('style', 'transform: rotate(10deg);');
        label.setAttribute('class', 'spin');
    }, 2000);
    
    setTimeout(function() {
        arm.setAttribute('style', 'transition-duration: 0s; transform: rotate(10deg);');
        allowNeedleGrab = true;
    }, 3000);
    
    getArmCircleCenter();
    
    window.onscroll = function() {
        getArmCircleCenter();
    }
    
    window.onresize = function() {
        getArmCircleCenter();
    }

    window.onmouseup = function() {
        if (needleUp) {
            needleUp = false;
            
            if (currentTrack === 'Stopped') {
                moveNeedle(0);
                
                label.addEventListener('animationiteration', stopLabelSpin);
                label.addEventListener('webkitAnimationIteration', stopLabelSpin);
            }
        }
        
        releaseStopButton();
    }
    
    window.addEventListener('touchmove', handleMousemove);
    
    window.ontouchend = function() {
        if (needleUp) {
            needleUp = false;
            
            if (currentTrack === 'Stopped') {
                moveNeedle(0);
                
                label.addEventListener('animationiteration', stopLabelSpin);
                label.addEventListener('webkitAnimationIteration', stopLabelSpin);
            }
        }
    }
});

let needleUp = false;
let currentTrack = 'Stopped';
let currentMousePosition = { x: 0, y: 0 };
let currentTouchPosition = { x: 0, y: 0 };
let angle = 0;

function getArmCircleCenter() {
    armCircleDimensions = armCircle.getBoundingClientRect();
    
    scrollPositionY = window.pageYOffset;
    window.scrollTo(0, 0);
    
    armCircleRadius = armCircleDimensions.width / 2;
    armCircleCenter.x = armCircleDimensions.left + armCircleRadius;
    armCircleCenter.y = armCircleDimensions.top + armCircleRadius;
    
    window.scrollTo(0, scrollPositionY);
}

function handleMousemove() {
    setAngle(event);
}

function setTrackNumber(angle) {
    if (angle > 30) {
        let values = Object.values(angles);
        values.push(angle);
        let angleIndex = values.sort().indexOf(angle) - 1;
        let key = Object.keys(angles)[angleIndex];
        
        currentTrack = key;
    } else {
        currentTrack = 'Stopped';
    }
}

function grabNeedle(event) {
    if (allowNeedleGrab) {
        needleUp = true;
        
        startLabelSpin();
    }
}

function moveNeedle(angle) {
    allowNeedleGrab = false;
    
    arm.setAttribute('style', 'transition-duration: 1s; transform: rotate(' + angle + 'deg);');
    
    setTimeout(function() {
        allowNeedleGrab = true;
        
        arm.setAttribute('style', 'transition-duration: 0s; transform: rotate(' + angle + 'deg);');
    }, 1000);
}

function stopPlaying() {
    stopButton.setAttribute('y', '257');
    currentTrack = 'Stopped';
    
    moveNeedle(0);
    
    label.addEventListener('animationiteration', stopLabelSpin);
    label.addEventListener('webkitAnimationIteration', stopLabelSpin);
}

function releaseStopButton() {
    stopButton.setAttribute('y', '253');
}

function clickTrack(event) {
    if (allowNeedleGrab) {
        currentTrack = event.target.getAttribute('data-track');
        
        startLabelSpin();
        
        arm.setAttribute('style', 'transition-duration: 1s; transform: rotate(' + angles[currentTrack] + 'deg);');
    }
}

function startLabelSpin() {
    label.removeEventListener('animationiteration', stopLabelSpin);
    label.removeEventListener('webkitAnimationIteration', stopLabelSpin);
    label.setAttribute('class', 'spin');
}

let target = '';

function touchNeedle(event) {
    target = event.target;
    
    needleUp = true;
    
    startLabelSpin();
    
    target.addEventListener('touchmove', function(e) {
        e.touches[0].clientX;
        
        setAngle(e.touches[0]);
    });
}

let lastAngle = 10;

function setAngle(event) {
    if (needleUp && allowNeedleGrab) {
        currentMousePosition.x = event.clientX;
        currentMousePosition.y = event.clientY;
        
        let	distanceX = currentMousePosition.x - armCircleCenter.x,
            distanceY = currentMousePosition.y - armCircleCenter.y;
            
        let angle = 0;
        
        if (window.innerWidth > window.innerHeight) {
            angle = ((Math.atan2(distanceY, distanceX)) * (180 / Math.PI)) - 90;
        } else {
            angle = ((Math.atan2(distanceX, distanceY)) * (180 / Math.PI)) - 90;
        }
        
        if (!Number.isNaN(angle)) {
            var lessThanTen = Math.abs(angle - lastAngle) < 5;
            
            if (lessThanTen) {
                setTrackNumber(angle);
                
                if (angle >= 0 && angle <= 48) {
                    arm.setAttribute('style', 'transform: rotate(' + angle + 'deg); transition-duration: 0s;');
                }
                
                lastAngle = angle;
            }
        }
    }
}
</script>

<style>
    .track {
        cursor: pointer;
    }
    
    #turn {
        transform-origin: 50% 0%;
        transform-box: fill-box;
        transition-duration: 1s;
    }
</style>

<div class="wrapper">
    <div id='turntableContainer' class='mx-auto max-w-6xl' on:mousemove={handleMousemove} on:touchstart={touchNeedle}>
        <svg viewBox='0 0 450 308' fill='none' xmlns='http://www.w3.org/2000/svg'>
            <g id='recordPlayer'>
                <g id='table'>
                    <rect id='top' y='13' width='450' height='275' class='fill-current text-white'/>
                    <rect id='front' y='288' width='450' height='20' class='fill-current text-gray-200'/>
                </g>
                <circle id='record' cx='200' cy='150' r='150' class='fill-current text-gray-900'/>
                <circle on:click={clickTrack} data-track='One' id='trackOne' cx='200.5' cy='150.5' r='132.5' class='track stroke-current text-gray-800' stroke-width='10'/>
                <circle on:click={clickTrack} data-track='Two' id='trackTwo' cx='200.5' cy='150.5' r='117.5' class='track stroke-current text-gray-800' stroke-width='10'/>
                <circle on:click={clickTrack} data-track='Three' id='trackThree' cx='200.5' cy='150.5' r='102.5' class='track stroke-current text-gray-800' stroke-width='10'/>
                <circle on:click={clickTrack} data-track='Four' id='trackFour' cx='200.5' cy='150.5' r='87.5' class='track stroke-current text-gray-800' stroke-width='10'/>
                <circle on:click={clickTrack} data-track='Five' id='trackFive' cx='200.5' cy='150.5' r='72.5' class='track stroke-current text-gray-800' stroke-width='10'/>
                <mask id='labelMask' mask-type='alpha' maskUnits='userSpaceOnUse' x='150' y='100' width='100' height='100'>
                    <circle cx='200' cy='150' r='50' class='fill-current text-teal-600'/>
                </mask>
                <g id='label' transform-origin='200px 150px'>
                    <circle id='labelColor' cx='200' cy='150' r='50' class='fill-current text-teal-600'/>
                    <g mask='url(#labelMask)'>
                        <g id='avatar'>
                            <g id='clothes'>
                                <path id='shirt' d='M208.814 199.998L223.854 181.3C223.854 175.652 212.422 171.02 206.774 171.02H193.248C187.542 171.02 177.252 175.595 177.252 181.3L192.19 199.998H208.814Z' fill='#E6E7E8'/>
                                <path id='rightCardigan' d='M208.263 171.133H213.686C219.391 171.133 223.967 175.765 223.967 181.413L223.912 200H208.263V171.133Z' fill='#5EC5C3'/>
                                <path id='leftCardigan' d='M192.955 171.133H187.532C181.827 171.133 177.252 175.765 177.252 181.413V199.995H192.955V171.133Z' fill='#5EC5C3'/>
                                <path id='collar' d='M210.636 179.832L209.449 170.511L191.261 170.172L190.074 179.832L200.637 177.233L210.636 179.832Z' fill='white'/>
                            </g>
                            <g id='neckWithShadow'>
                                <path id='neck' d='M192.277 152.04V167.009V167.969V171.076L200.355 176.329L208.489 171.076V170.003V168.29V152.04H192.277Z' fill='#FCCEAE'/>
                                <path id='neckShadow' d='M208.489 168.29L192.277 166.162V161.417H208.546L208.489 168.29Z' fill='#F2A87E'/>
                            </g>
                            <g id='ears'>
                                <path id='rightEarOuter' d='M225.259 139.999C225.963 136.864 223.991 133.752 220.856 133.049C217.721 132.345 214.609 134.317 213.906 137.452C213.202 140.587 215.174 143.699 218.309 144.402C221.444 145.106 224.556 143.134 225.259 139.999Z' fill='#FCCEAE'/>
                                <path id='rightEarInner' d='M221.877 138.709C221.877 139.952 220.86 140.969 219.617 140.969C218.374 140.969 217.358 139.952 217.358 138.709C217.358 137.466 218.374 136.45 219.617 136.45C220.86 136.45 221.877 137.466 221.877 138.709Z' fill='#F2A87E'/>
                                <path id='leftEarOuter' d='M187.101 140.038C187.81 136.904 185.843 133.789 182.709 133.08C179.575 132.371 176.459 134.337 175.751 137.471C175.042 140.605 177.008 143.721 180.142 144.43C183.276 145.139 186.392 143.172 187.101 140.038Z' fill='#FCCEAE'/>
                                <path id='leftEarInner' d='M183.748 138.709C183.748 139.952 182.731 140.969 181.488 140.969C180.246 140.969 179.229 139.952 179.229 138.709C179.229 137.466 180.246 136.45 181.488 136.45C182.731 136.45 183.748 137.466 183.748 138.709Z' fill='#F2A87E'/>
                            </g>
                            <g id='face'>
                                <path d='M219.617 131.253V143.454H219.561V145.714C219.561 155.994 211.2 164.185 200.92 164.185H200.073C189.792 164.185 181.432 155.994 181.432 145.714V132.891L181.488 132.892V131.253C181.488 120.972 189.848 112.612 200.129 112.612H200.976C211.257 112.612 219.617 120.972 219.617 131.253Z' fill='#FCCEAE'/>
                            </g>
                            <g id='eyes'>
                                <path id='rightEye' d='M210.636 136.393C210.636 135.207 209.675 134.303 208.546 134.303H208.489C207.303 134.303 206.399 135.263 206.399 136.393H210.636Z' fill='#343333'/>
                                <path id='leftEye' d='M195.667 136.393C195.667 135.207 194.706 134.303 193.577 134.303H193.52C192.334 134.303 191.43 135.263 191.43 136.393H195.667Z' fill='#343333'/>
                            </g>
                            <g id='eyebrows'>
                                <path id='rightEyebrow' d='M207.472 133.399H212.217C212.217 131.987 211.087 130.857 209.675 130.857H204.93C204.93 132.27 206.06 133.399 207.472 133.399Z' fill='#E27504'/>
                                <path id='leftEyebrow' d='M194.706 133.399H189.961C189.961 131.987 191.091 130.857 192.503 130.857H197.248C197.248 132.27 196.118 133.399 194.706 133.399Z' fill='#E27504'/>
                            </g>
                            <g id='mouthWithShadow'>
                                <path id='lips' d='M207.642 146.787C206.738 149.724 203.97 151.814 200.75 151.814C197.587 151.814 194.876 149.724 193.915 146.9L207.642 146.787Z' fill='#F2A87E'/>
                                <path id='teeth' d='M200.75 150.176C198.152 150.232 195.836 148.933 194.424 146.956L206.907 146.73C205.608 148.707 203.349 150.12 200.75 150.176Z' fill='white'/>
                                <path id='mouthShadow' opacity='0.2' d='M206.06 147.747L195.271 147.973C195.045 147.747 194.593 147.182 194.367 146.9L206.851 146.674C206.794 147.069 206.286 147.521 206.06 147.747Z' fill='#201600'/>
                            </g>
                            <g id='beard'>
                                <path id='rightBeard' d='M214.138 160.796L214.194 163.168C217.414 163.168 219.561 160.57 219.561 157.35L219.63 140.634L218.012 140.411C215.244 140.411 214.138 142.607 214.138 145.375V160.796Z' fill='#E27504'/>
                                <path id='leftBeard' d='M186.629 160.796L186.572 163.168C183.352 163.168 181.206 160.57 181.206 157.35L181.262 140.622L182.11 140.404C184.878 140.404 186.629 142.607 186.629 145.375V160.796Z' fill='#E27504'/>
                                <path id='chin' d='M215.889 166.218H184.991C182.957 166.218 181.262 164.524 181.262 162.49V157.689C181.262 155.655 182.957 153.961 184.991 153.961H215.889C217.922 153.961 219.617 155.655 219.617 157.689V162.49C219.617 164.524 217.922 166.218 215.889 166.218Z' fill='#E99112'/>
                            </g>
                            <g id='mustache'>
                                <path d='M188.21 147.747C188.21 145.996 189.001 143.736 191.374 143.736H212.726V145.092C212.726 146.561 211.935 147.634 209.845 147.747H188.21V147.747Z' fill='#E27504'/>
                            </g>
                            <g id='nose'>
                                <path d='M203.621 143.172H204.591C205.043 143.172 205.382 142.833 205.326 142.437C205.326 142.042 204.987 141.703 204.591 141.703H196.909C196.514 141.703 196.175 142.042 196.175 142.437C196.175 142.833 196.514 143.172 196.909 143.172H197.891C198.4 144.109 199.395 144.753 200.524 144.753H200.976C202.145 144.753 203.125 144.109 203.621 143.172Z' fill='#EE9A64'/>
                            </g>
                            <g id='hair'>
                                <path id='rightSideburn' d='M219.628 140.635L219.561 140.63C216.341 140.63 213.742 138.031 213.742 134.812V121.085H219.617L219.628 140.635Z' fill='#E99112'/>
                                <path id='rightPart' d='M219.617 128.937H219.504C213.855 128.937 209.28 124.361 209.28 118.713V111.144H209.393C215.042 111.144 219.617 115.719 219.617 121.368V128.937Z' fill='#E27504'/>
                                <path id='leftPart' d='M198.999 107.359C208.715 107.359 211.596 112.16 212.048 116.849C212.029 116.872 212.04 116.993 212.058 117.192C212.251 119.339 213.269 130.649 187.137 128.985V134.812C187.137 138.031 184.539 140.63 181.319 140.63H181.262V124.587C181.262 124.523 181.264 124.459 181.266 124.396C181.459 114.959 189.178 107.359 198.66 107.359H198.999Z' fill='#E99112'/>
                                <g id='spots'>
                                    <path id='leftSpot' d='M195.78 123.627C197.152 123.627 198.265 122.514 198.265 121.142C198.265 119.769 197.152 118.656 195.78 118.656C194.407 118.656 193.294 119.769 193.294 121.142C193.294 122.514 194.407 123.627 195.78 123.627Z' fill='#F0AD21'/>
                                    <path id='rightSpot' d='M190.865 124.644C191.583 124.644 192.164 124.062 192.164 123.345C192.164 122.627 191.583 122.045 190.865 122.045C190.148 122.045 189.566 122.627 189.566 123.345C189.566 124.062 190.148 124.644 190.865 124.644Z' fill='#F0AD21'/>
                                </g>
                            </g>
                            <g id='glasses'>
                                <path id='leftReflection' opacity='0.5' d='M193.577 130.067L184.087 140.468C184.087 140.468 183.63 138.263 183.686 135.383C183.742 132.502 184.143 130.01 184.143 130.01L193.577 130.067Z' fill='white'/>
                                <path id='rightReflection' opacity='0.5' d='M213.234 130.01L205.382 140.912C205.382 140.912 204.196 139.161 203.688 137.692C203.123 136.224 202.784 134.755 202.784 134.755L203.857 130.123L213.234 130.01Z' fill='white'/>
                                <path id='frames' fill-rule='evenodd' clip-rule='evenodd' d='M197.981 129.564L183.668 129.383L183.587 129.858L184.143 129.954C183.587 129.858 183.586 129.859 183.586 129.859L183.585 129.866L183.582 129.886L183.569 129.965C183.557 130.034 183.541 130.135 183.52 130.263C183.48 130.52 183.423 130.888 183.361 131.333C183.235 132.222 183.084 133.423 182.98 134.666C182.877 135.904 182.82 137.204 182.888 138.283C182.923 138.821 182.99 139.326 183.107 139.749C183.219 140.154 183.401 140.574 183.73 140.842C184.007 141.068 184.4 141.231 184.816 141.357C185.245 141.486 185.755 141.592 186.312 141.676C187.427 141.845 188.771 141.932 190.108 141.936C191.445 141.94 192.795 141.859 193.92 141.685C194.482 141.598 195 141.485 195.438 141.343C195.865 141.205 196.266 141.023 196.551 140.769L196.555 140.765L196.559 140.762C197.012 140.342 197.39 139.748 197.702 139.133C198.017 138.511 198.283 137.829 198.496 137.204C198.608 136.875 198.707 136.558 198.791 136.27C199.054 135.522 199.816 134.981 200.715 134.981C201.582 134.981 202.346 135.523 202.597 136.275C202.682 136.561 202.78 136.876 202.892 137.204C203.105 137.829 203.371 138.511 203.686 139.133C203.997 139.747 204.376 140.342 204.829 140.761C205.114 141.026 205.517 141.215 205.945 141.358C206.384 141.505 206.904 141.62 207.467 141.708C208.593 141.886 209.944 141.964 211.281 141.957C212.619 141.95 213.963 141.857 215.078 141.684C215.635 141.598 216.145 141.49 216.574 141.359C216.989 141.231 217.381 141.067 217.658 140.842C217.986 140.574 218.169 140.154 218.281 139.749C218.398 139.326 218.465 138.821 218.499 138.283C218.568 137.204 218.511 135.904 218.408 134.666C218.304 133.423 218.153 132.222 218.027 131.333C217.964 130.888 217.908 130.52 217.867 130.263C217.847 130.135 217.831 130.034 217.819 129.965L217.806 129.886L217.803 129.866L217.801 129.859L217.801 129.858L217.72 129.383L203.407 129.564L202.314 134.28C201.838 134.004 201.286 133.851 200.715 133.851C200.121 133.851 199.558 134.013 199.077 134.296L197.981 129.564ZM217.245 129.954L217.801 129.858C217.801 129.858 217.801 129.858 217.245 129.954ZM184.479 131.491C184.534 131.1 184.584 130.77 184.623 130.525L197.08 130.682L198.024 134.756L198.012 134.807C197.986 134.921 197.945 135.085 197.892 135.287C197.786 135.692 197.629 136.245 197.427 136.839C197.223 137.435 196.977 138.064 196.694 138.622C196.408 139.186 196.103 139.642 195.795 139.929C195.684 140.027 195.459 140.149 195.09 140.269C194.728 140.386 194.273 140.487 193.747 140.568C192.696 140.731 191.408 140.81 190.111 140.806C188.815 140.803 187.528 140.717 186.481 140.559C185.957 140.48 185.503 140.384 185.143 140.275C184.769 140.162 184.547 140.05 184.444 139.966C184.391 139.923 184.289 139.783 184.196 139.447C184.107 139.129 184.048 138.71 184.016 138.211C183.952 137.214 184.004 135.978 184.106 134.76C184.207 133.545 184.356 132.367 184.479 131.491ZM203.363 134.756L203.375 134.807C203.402 134.921 203.442 135.085 203.495 135.287C203.602 135.692 203.758 136.245 203.961 136.839C204.164 137.435 204.411 138.064 204.694 138.622C204.981 139.189 205.287 139.646 205.597 139.933L205.597 139.933C205.708 140.036 205.931 140.162 206.303 140.287C206.663 140.407 207.117 140.51 207.642 140.592C208.692 140.758 209.979 140.834 211.275 140.827C212.571 140.82 213.857 140.73 214.905 140.568C215.429 140.486 215.883 140.388 216.243 140.278C216.617 140.164 216.84 140.051 216.944 139.966C216.997 139.923 217.099 139.783 217.192 139.447C217.28 139.129 217.34 138.71 217.372 138.211C217.436 137.214 217.383 135.978 217.282 134.76C217.181 133.545 217.032 132.367 216.909 131.491C216.853 131.1 216.803 130.77 216.765 130.525L204.307 130.682L203.363 134.756Z' fill='#343333'/>
                            </g>
                        </g>
                    </g>
                </g>
                <g id='stop' on:mousedown={stopPlaying} class='cursor-pointer'>
                    <rect x='20' y='273' width='50' height='4' class='fill-current text-gray-900'/>
                    <rect id='stopButton' x='20' y='253' width='50' height='20' class='fill-current text-gray-700'/>
                </g>
                <g id='arm'>
                    <circle id='armCircle' cx='415' cy='48' r='15' class='fill-current text-gray-400'/>
                    <g id='turn' style="transition-duration: 2s;">
                        <rect x='410' y='48' width='10' height='200' class='fill-current text-gray-400'/>
                        <g id='needle' on:mousedown={grabNeedle} class='cursor-move'>
                            <rect x='405' y='248' width='17' height='30' class='fill-current text-gray-900'/>
                            <rect x='422' y='248' width='3' height='30' class='fill-current text-red-600'/>
                        </g>
                    </g>
                </g>
            </g>
        </svg>
    </div>
</div>