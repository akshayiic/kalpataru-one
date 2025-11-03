<script>
	import { getContext, onDestroy } from 'svelte';
	import * as Accordion from '$lib/components/ui/accordion/index.ts';
	import minimizeBtn from '$lib/images/minimize-icon.svg';
	import maximizeBtn from '$lib/images/maximize-icon.svg';
	import { page } from '$app/stores';
	import { setContext } from 'svelte';
	import { writable } from 'svelte/store';
	let hotspotName = getContext('hotspotName');
	import { goto } from '$app/navigation';
	import bowser from 'bowser';
	let unsubscribeHotSpot;
	import { onMount } from 'svelte';
	import { browser } from '$app/environment';
	let Marzipano;
	const isAmenitiesMinimized = writable();
	const ishighlights = writable();
	const isSafariMobile =
		browser &&
		/iPhone|iPad|iPod/.test(navigator.userAgent) &&
		/Safari/.test(navigator.userAgent) &&
		!/CriOS|FxiOS|OPiOS|EdgiOS/.test(navigator.userAgent);

	setContext('isAmenitiesMinimized', isAmenitiesMinimized);
	$: isAmenitiesMinimized.set(isSafariMobile ? true : false);
	$ishighlights = false;
	onMount(async () => {
		$hotspotName = '0-entrance-1';

		// console.log('opkasease yolo thius');
		const url = window.location.href;
		if (url.includes('podium')) {
			$hotspotName = '0-lappool';
		}

		// @ts-expect-error because my dead marzipano doesn't have type bindings like i even cared in the first place
		Marzipano = await import('marzipano');
		let allScenes = {};
		function resetNegativeTranslations(inputString, element) {
			// Regular expression to match translate values
			const translateRegex = /translate[XYZ]?\((-?\d+(\.\d+)?px)\)/g;

			// Replace negative or large translate values
			const transformedString = inputString.replace(translateRegex, (match, value) => {
				const numericValue = parseFloat(value);
				if (numericValue < 0) {
					setTimeout(() => {
						if (match.charAt(9) == 'X') {
							element.classList.add('left-arrow');
						} else {
							element.classList.add('top-arrow');
						}
					}, 200);

					return `translate${match.charAt(9)}(100px)`;
				} else if (match.includes('translateX') && numericValue > window.innerWidth) {
					setTimeout(() => {
						element.classList.add('right-arrow');
					}, 200);

					return `translateX(${window.innerWidth - 250}px)`;
				} else if (match.includes('translateY') && numericValue > window.innerHeight) {
					setTimeout(() => {
						element.classList.add('down-arrow');
					}, 200);

					return `translateY(${window.innerHeight - 200}px)`;
				}
				element.classList.remove('left-arrow');
				element.classList.remove('right-arrow');
				element.classList.remove('down-arrow');
				element.classList.remove('top-arrow');
				return match;
			});

			return transformedString;
		}

		// Create viewer.
		var viewer = new Marzipano.Viewer(document.getElementById('pano'), {
			controls: {
				mouseViewMode: 'drag' // drag|qtvr
			}
		});
		window.dragControlMethod = bowser.mobile
			? viewer.controls().method('touchView').instance
			: viewer.controls().method('mouseViewDrag').instance;
		window.Mzviewer = viewer;
		console.log('Marzipano initialized main app', window?.dragControlMethod);

		let counter = 0;

		const createHotspot = (
			scene,
			position,
			nextScene,
			tooltipText = 'Area Label',
			hotspotNameUpdate
		) => {
			// Create wrapper element to hold icon and tooltip.
			var wrapper = document.createElement('div');

			wrapper.classList.add('info-hotspot');
			wrapper.id = 'hotspot-wrapper-' + tooltipText.replaceAll(' ', '') + counter;

			// hotspots
			var imgHotspot = document.createElement('div');
			imgHotspot.innerText = tooltipText;
			imgHotspot.id = 'hotspot-' + tooltipText.replaceAll(' ', '') + counter;
			counter++;
			imgHotspot.classList.add('hotspot');

			// imgHotspot.classList.add('animate-bounce');
			// imgHotspot.style.transform = transforms;

			wrapper.appendChild(imgHotspot);

			imgHotspot.addEventListener('click', function () {
				nextScene.switchTo();
				console.log('next scene data', nextScene);
				window.view = nextScene._view;
				// Create text element.
				$hotspotName = hotspotNameUpdate;

				console.log('hotspot name changed', hotspotNameUpdate);
			});

			scene.hotspotContainer().createHotspot(wrapper, position, {});
		};

		const create360scene = (name, size, initialView) => {
			var urlPrefix = 'https://assets.vestate.io/kalpataru-one/amenities';

			// const urlPrefix = '/render/amenities';
			// let source = Marzipano.ImageUrlSource.fromString(`${urlPrefix}/${name}/{z}/{f}/{y}/{x}.jpg`, {
			// 	cubeMapPreviewUrl: `${urlPrefix}/${name}/preview.jpg`
			// });

			let source = Marzipano.ImageUrlSource.fromString(
				urlPrefix + '/' + name + '/{z}/{f}/{y}/{x}.jpg',
				{ cubeMapPreviewUrl: urlPrefix + '/' + name + '/preview.jpg' }
			);
			var geometry = new Marzipano.CubeGeometry(size);

			// let source = new Marzipano.ImageUrlSource.fromString(path);
			var limiter = Marzipano.RectilinearView.limit.traditional(3840, (130 * Math.PI) / 180);

			var view = new Marzipano.RectilinearView(initialView, limiter);

			let scene = viewer.createScene({
				source: source,
				geometry: geometry,
				view: view,
				pinFirstLevel: true
			});
			allScenes[name] = {
				source,
				view,
				scene
			};
		};

		create360scene(
			'0-entrance-1',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				},
				{
					tileSize: 512,
					size: 4096
				}
			],
			{
				yaw: 2.632707634607316,
				pitch: -0.003119767609573998,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'1-entrance-2',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				},
				{
					tileSize: 512,
					size: 4096
				}
			],
			{
				yaw: -1.3010636242139597,
				pitch: -0.0006718854569758292,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'2-entrance-3',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				},
				{
					tileSize: 512,
					size: 4096
				}
			],
			{
				yaw: -1.3769785749493764,
				pitch: -0.07663311220942148,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'3-drop-off',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -0.5489709503436515,
				pitch: -0.0014714328772225116,
				fov: 1.3966064861122105
			}
		);

		create360scene(
			'4-entrance-lobby',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 2.1624774425860824,
				pitch: -0.00407508819588287,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'5-lift-lobby',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 1.8352486450153673,
				pitch: -0.012324038513211732,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'6-convinience-store',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -0.6094501063720816,
				pitch: -0.04901920108497926,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'7-jogging-path',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 1.4745777751684983,
				pitch: -0.012054507759323485,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'8-garden-pavillion',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 2.653041993680091,
				pitch: 0.11412806779712525,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'9-kids-play-area',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 0.7053111780405867,
				pitch: -0.004275535355620974,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'10-box-cricket',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -1.0970323896160163,
				pitch: -0.01928349555596398,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'11-social-pods',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -0.8135085087819025,
				pitch: 0.0011535142441534418,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'12-jogging-track---500m',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 0.01132105460752797,
				pitch: -0.014788846215854079,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'13-banquet-drop-off',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 2.2071864507330226,
				pitch: 0.02711288472906581,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'14-banquet-hall',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -0.7867670833961444,
				pitch: 0.0073944231079270395,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'15-pet-run',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 0.8740485229695363,
				pitch: 0.06057475995799244,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'0-activity-room',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -2.4678218838298136,
				pitch: 0.018867516201112267,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'1-gymnasium-cardio',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 0.6712083244368028,
				pitch: 0.04825945101113405,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'2-juice-bar',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 0.6692150196461473,
				pitch: -0.00444808275332953,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'3-business-centre',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -2.4387183590144303,
				pitch: 0.027695184032790365,
				fov: 1.4806022199767128
			}
		);

		create360scene(
			'4-indoor-kids-play-area',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 1.1994558832391036,
				pitch: -0.00332105521830961,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'5-salon',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 0.24670507199383174,
				pitch: 0.02943270374331952,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'6-mini-theatre',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				},
				{
					tileSize: 512,
					size: 4096
				}
			],
			{
				yaw: -2.4276800887936876,
				pitch: -0.03383508755444886,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'7-aqua-deck',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 1.1566344124027275,
				pitch: 0.009859230810569386,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'8-event-deck-lawn',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -2.134692048253461,
				pitch: 0.0284685332789536,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'9-family-pavilion',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 2.009699255290382,
				pitch: -0.0018237910134697444,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'10-fitness-corner',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 3.027458658817423,
				pitch: -0.026139123202987946,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'11-kids-pool',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -2.7909975563495504,
				pitch: -0.0333350041596816,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'12-lap-pool',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				},
				{
					tileSize: 512,
					size: 4096
				}
			],
			{
				yaw: 1.7047269759476116,
				pitch: 0.05423283635339793,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'13-landscape-garden',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -1.9351640850581973,
				pitch: -0.0032203339429113242,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'14-multipurpose-lawn',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 0.8458517590839705,
				pitch: 0.040308082508625276,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'15-spa-pod',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 0.21083104626045035,
				pitch: 0.022929191928756865,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'16-walking-track',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -1.5583098695071804,
				pitch: -0.0026097963910327593,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'17-kids-play-zone',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 1.0451442512547597,
				pitch: 0.025961606113000357,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'18-social-pods',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 1.9132520449907844,
				pitch: 0.04860215154732472,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'0-multipurpose-court',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -1.297193576625295,
				pitch: 0.0073944231079270395,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'1-cards-room',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 0.2862207519519231,
				pitch: -0.009316189193633306,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'2-games-room',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 0.23938866565547912,
				pitch: -0.020355393556556578,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'3-gymnasium---weights',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 0.23111580402341758,
				pitch: -0.10965096310635758,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'4-jacuzzi-lounge',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 2.2048096863599156,
				pitch: -0.0073944231079270395,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'5-mens-spa',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -1.310079636626643,
				pitch: 0.2950374325654348,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'6-squash-court',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 0.1905710858934455,
				pitch: 0.05624243030573339,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'7-yoga-area',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -2.461319983591558,
				pitch: -0.022389418435878383,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'8-womens-spa',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				},
				{
					tileSize: 512,
					size: 4096
				}
			],
			{
				yaw: -1.3330312237513393,
				pitch: 0.5575009250423868,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'0-cozy-lounge-deck-tower-a',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -1.1249341318500345,
				pitch: -0.037467286446872805,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'1-sky-lounge-room-1-tower-a',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 1.7519349754714746,
				pitch: 0.034092097760737516,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'2-sky-lounge-room-2-tower-a',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -0.2755356573849035,
				pitch: 0.046810175950053434,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'3-lap-pool-tower-a',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -2.411606612866951,
				pitch: -0.04248168569848332,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'4-chillout-lawn-tower-a',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -1.3410960673027272,
				pitch: 0.04929615405284693,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'5-seating-planters-tower-b',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -2.0270564913932176,
				pitch: -0.06933757300159726,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'6-chillout-deck-tower-b',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -2.3852316149001496,
				pitch: 0.0149221908821211,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'7-lap-pool-tower-b',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 1.1734881915178512,
				pitch: 0.01233887152107549,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'8-social-deck-tower-b',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 2.5961077678054068,
				pitch: 0.01670425381999685,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'9-lap-pool-tower-c',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 0.7056790705360889,
				pitch: -0.009859230810569386,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'10-chillout-deck-tower-c',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: -2.2509902766969496,
				pitch: -0.015112753111624855,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'11-pool-loungers-tower-c',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 0.8322646719897868,
				pitch: -0.025629583004192824,
				fov: 1.5552936603672898
			}
		);

		create360scene(
			'12-lookout-deck-tower-c',
			[
				{
					tileSize: 256,
					size: 256,
					fallbackOnly: true
				},
				{
					tileSize: 512,
					size: 512
				},
				{
					tileSize: 512,
					size: 1024
				},
				{
					tileSize: 512,
					size: 2048
				}
			],
			{
				yaw: 1.4381840220269915,
				pitch: 0.019438988205259022,
				fov: 1.5552936603672898
			}
		);

		// 0-cozy-lounge-deck-tower-a hotspots
		createHotspot(
			allScenes['0-cozy-lounge-deck-tower-a'].scene,
			{ yaw: 2.5995735939091844, pitch: 0.4498619755502897 },
			allScenes['7-lap-pool-tower-b'].scene,
			'Lap Pool Tower B',
			'7-lap-pool-tower-b'
		);

		createHotspot(
			allScenes['0-cozy-lounge-deck-tower-a'].scene,
			{ yaw: 2.998623718874051, pitch: 0.4552501065285455 },
			allScenes['6-chillout-deck-tower-b'].scene,
			'Chill Out Deck Tower B',
			'6-chillout-deck-tower-b'
		);

		createHotspot(
			allScenes['0-cozy-lounge-deck-tower-a'].scene,
			{ yaw: 2.1741902712875483, pitch: 0.4963232932167081 },
			allScenes['10-chillout-deck-tower-c'].scene,
			'Chill Out Deck Tower C',
			'10-chillout-deck-tower-c'
		);

		// 1-sky-lounge-room-1-tower-a hotspots
		createHotspot(
			allScenes['1-sky-lounge-room-1-tower-a'].scene,
			{ yaw: -2.6384563600506343, pitch: 0.04045849726684558 },
			allScenes['0-cozy-lounge-deck-tower-a'].scene,
			'Cozy Lounge Deck Tower A',
			'0-cozy-lounge-deck-tower-a'
		);

		createHotspot(
			allScenes['1-sky-lounge-room-1-tower-a'].scene,
			{ yaw: -2.9835315629191843, pitch: 0.10906783226639583 },
			allScenes['6-chillout-deck-tower-b'].scene,
			'Chill Out Deck Tower B',
			'6-chillout-deck-tower-b'
		);

		createHotspot(
			allScenes['1-sky-lounge-room-1-tower-a'].scene,
			{ yaw: 3.076531495949295, pitch: 0.1059073624805329 },
			allScenes['10-chillout-deck-tower-c'].scene,
			'Chill Out Deck Tower C',
			'10-chillout-deck-tower-c'
		);

		// 2-sky-lounge-room-2-tower-a hotspots
		createHotspot(
			allScenes['2-sky-lounge-room-2-tower-a'].scene,
			{ yaw: -2.329059474671187, pitch: -0.025939829276868664 },
			allScenes['0-cozy-lounge-deck-tower-a'].scene,
			'Cozy Lounge Deck Tower A',
			'0-cozy-lounge-deck-tower-a'
		);

		createHotspot(
			allScenes['2-sky-lounge-room-2-tower-a'].scene,
			{ yaw: 2.691950372628238, pitch: 0.12821425547538112 },
			allScenes['8-social-deck-tower-b'].scene,
			'Social Deck Tower B',
			'8-social-deck-tower-b'
		);

		createHotspot(
			allScenes['2-sky-lounge-room-2-tower-a'].scene,
			{ yaw: 2.483527263479825, pitch: 0.26098414549466753 },
			allScenes['10-chillout-deck-tower-c'].scene,
			'Chill Out Deck Tower C',
			'10-chillout-deck-tower-c'
		);

		// 3-lap-pool-tower-a hotspots
		createHotspot(
			allScenes['3-lap-pool-tower-a'].scene,
			{ yaw: 0.3812850471523639, pitch: 0.3168563177487531 },
			allScenes['5-seating-planters-tower-b'].scene,
			'Seating Planters Tower B',
			'5-seating-planters-tower-b'
		);

		createHotspot(
			allScenes['3-lap-pool-tower-a'].scene,
			{ yaw: 0.18660414570833872, pitch: 0.3491787621364537 },
			allScenes['9-lap-pool-tower-c'].scene,
			'Lap Pool Tower C',
			'9-lap-pool-tower-c'
		);

		createHotspot(
			allScenes['3-lap-pool-tower-a'].scene,
			{ yaw: 0.6369424285848808, pitch: 0.3624097141435403 },
			allScenes['11-pool-loungers-tower-c'].scene,
			'Pool Loungers Tower C',
			'11-pool-loungers-tower-c'
		);

		// 4-chillout-lawn-tower-a hotspots
		createHotspot(
			allScenes['4-chillout-lawn-tower-a'].scene,
			{ yaw: 0.5919200233001849, pitch: 0.19843872562532638 },
			allScenes['3-lap-pool-tower-a'].scene,
			'Lap Pool Tower A',
			'3-lap-pool-tower-a'
		);

		createHotspot(
			allScenes['4-chillout-lawn-tower-a'].scene,
			{ yaw: 0.46625375583662176, pitch: -0.006390223917099647 },
			allScenes['6-chillout-deck-tower-b'].scene,
			'Chill Out Deck Tower B',
			'6-chillout-deck-tower-b'
		);

		createHotspot(
			allScenes['4-chillout-lawn-tower-a'].scene,
			{ yaw: 0.7318513353194351, pitch: -0.01561630195647723 },
			allScenes['10-chillout-deck-tower-c'].scene,
			'Chill Out Deck Tower C',
			'10-chillout-deck-tower-c'
		);

		// 5-seating-planters-tower-b hotspots
		createHotspot(
			allScenes['5-seating-planters-tower-b'].scene,
			{ yaw: -2.890873027905485, pitch: -0.9401895475321123 },
			allScenes['7-lap-pool-tower-b'].scene,
			'Lap Pool Tower B',
			'7-lap-pool-tower-b'
		);

		createHotspot(
			allScenes['5-seating-planters-tower-b'].scene,
			{ yaw: -2.1839388587900643, pitch: -0.4927979442542796 },
			allScenes['6-chillout-deck-tower-b'].scene,
			'Chill Out Deck Tower B',
			'6-chillout-deck-tower-b'
		);

		// 6-chillout-deck-tower-b hotspots
		createHotspot(
			allScenes['6-chillout-deck-tower-b'].scene,
			{ yaw: 0.7177602558524665, pitch: 0.22152091490884374 },
			allScenes['7-lap-pool-tower-b'].scene,
			'Lap Pool Tower B',
			'7-lap-pool-tower-b'
		);

		createHotspot(
			allScenes['6-chillout-deck-tower-b'].scene,
			{ yaw: 0.842097182734804, pitch: 0.050589942386469744 },
			allScenes['12-lookout-deck-tower-c'].scene,
			'Lookout Deck Tower C',
			'12-lookout-deck-tower-c'
		);

		createHotspot(
			allScenes['6-chillout-deck-tower-b'].scene,
			{ yaw: 0.6738871994208395, pitch: 0.005470394629760733 },
			allScenes['11-pool-loungers-tower-c'].scene,
			'Pool Loungers Tower C',
			'11-pool-loungers-tower-c'
		);

		// 7-lap-pool-tower-b hotspots
		createHotspot(
			allScenes['7-lap-pool-tower-b'].scene,
			{ yaw: 1.101818616459198, pitch: 0.021645343840258846 },
			allScenes['6-chillout-deck-tower-b'].scene,
			'Chill Out Deck Tower B',
			'6-chillout-deck-tower-b'
		);

		createHotspot(
			allScenes['7-lap-pool-tower-b'].scene,
			{ yaw: -2.1724339961233454, pitch: -0.020267815838534986 },
			allScenes['9-lap-pool-tower-c'].scene,
			'Lap Pool Tower C',
			'9-lap-pool-tower-c'
		);

		createHotspot(
			allScenes['7-lap-pool-tower-b'].scene,
			{ yaw: -1.788961724483105, pitch: 0.3115360830721077 },
			allScenes['12-lookout-deck-tower-c'].scene,
			'Lookout Deck Tower C',
			'12-lookout-deck-tower-c'
		);

		createHotspot(
			allScenes['7-lap-pool-tower-b'].scene,
			{ yaw: 1.049406378027875, pitch: -1.1633715171740882 },
			allScenes['3-lap-pool-tower-a'].scene,
			'Lap Pool Tower A',
			'3-lap-pool-tower-a'
		);

		// 8-social-deck-tower-b hotspots
		createHotspot(
			allScenes['8-social-deck-tower-b'].scene,
			{ yaw: 1.0332479692995982, pitch: 0.31276229289212054 },
			allScenes['7-lap-pool-tower-b'].scene,
			'Lap Pool Tower B',
			'7-lap-pool-tower-b'
		);

		createHotspot(
			allScenes['8-social-deck-tower-b'].scene,
			{ yaw: 1.121037209071602, pitch: -0.014433011190217115 },
			allScenes['6-chillout-deck-tower-b'].scene,
			'Chill Out Deck Tower B',
			'6-chillout-deck-tower-b'
		);

		createHotspot(
			allScenes['8-social-deck-tower-b'].scene,
			{ yaw: -2.118678132486206, pitch: 0.02616063586407691 },
			allScenes['9-lap-pool-tower-c'].scene,
			'Lap Pool Tower C',
			'9-lap-pool-tower-c'
		);

		// 9-lap-pool-tower-c hotspots
		createHotspot(
			allScenes['9-lap-pool-tower-c'].scene,
			{ yaw: 0.7959688299827032, pitch: -0.040890172638075484 },
			allScenes['11-pool-loungers-tower-c'].scene,
			'Pool Loungers Tower C',
			'11-pool-loungers-tower-c'
		);

		createHotspot(
			allScenes['9-lap-pool-tower-c'].scene,
			{ yaw: -2.0701110974594634, pitch: -0.03156503693467627 },
			allScenes['10-chillout-deck-tower-c'].scene,
			'Chill Out Deck Tower C',
			'10-chillout-deck-tower-c'
		);

		// 10-chillout-deck-tower-c hotspots
		createHotspot(
			allScenes['10-chillout-deck-tower-c'].scene,
			{ yaw: -2.452324220101133, pitch: 0.5272008600286604 },
			allScenes['9-lap-pool-tower-c'].scene,
			'Lap Pool Tower C',
			'9-lap-pool-tower-c'
		);

		createHotspot(
			allScenes['10-chillout-deck-tower-c'].scene,
			{ yaw: 0.7563154632714504, pitch: -0.004615764702098346 },
			allScenes['11-pool-loungers-tower-c'].scene,
			'Pool Loungers Tower C',
			'11-pool-loungers-tower-c'
		);

		// 11-pool-loungers-tower-c hotspots
		createHotspot(
			allScenes['11-pool-loungers-tower-c'].scene,
			{ yaw: -2.460297334082391, pitch: 0.18777531448524698 },
			allScenes['9-lap-pool-tower-c'].scene,
			'Lap Pool Tower C',
			'9-lap-pool-tower-c'
		);

		createHotspot(
			allScenes['11-pool-loungers-tower-c'].scene,
			{ yaw: -2.3543708707655515, pitch: -0.08333062055995555 },
			allScenes['10-chillout-deck-tower-c'].scene,
			'Chill Out Deck Tower C',
			'10-chillout-deck-tower-c'
		);

		createHotspot(
			allScenes['11-pool-loungers-tower-c'].scene,
			{ yaw: -2.406302100350292, pitch: -0.841988709936107 },
			allScenes['3-lap-pool-tower-a'].scene,
			'Lap Pool Tower A',
			'3-lap-pool-tower-a'
		);

		// 12-lookout-deck-tower-c hotspots
		createHotspot(
			allScenes['12-lookout-deck-tower-c'].scene,
			{ yaw: 0.08944932489711022, pitch: -0.7121619353679414 },
			allScenes['9-lap-pool-tower-c'].scene,
			'Lap Pool Tower C',
			'9-lap-pool-tower-c'
		);

		createHotspot(
			allScenes['12-lookout-deck-tower-c'].scene,
			{ yaw: 0.7017150528553451, pitch: -0.18317378677643958 },
			allScenes['11-pool-loungers-tower-c'].scene,
			'Pool Loungers Tower C',
			'11-pool-loungers-tower-c'
		);

		// 0-multipurpose-court hotspots
		createHotspot(
			allScenes['0-multipurpose-court'].scene,
			{ yaw: -1.971521571598915, pitch: -0.12252280931670079 },
			allScenes['2-games-room'].scene,
			'Games Room',
			'2-games-room'
		);

		createHotspot(
			allScenes['0-multipurpose-court'].scene,
			{ yaw: -1.324686510380456, pitch: -0.03570899892614676 },
			allScenes['3-gymnasium---weights'].scene,
			'Gymnasium - Weights',
			'3-gymnasium---weights'
		);

		// 1-cards-room hotspots
		createHotspot(
			allScenes['1-cards-room'].scene,
			{ yaw: -1.4858587932400873, pitch: -0.02714151128046538 },
			allScenes['2-games-room'].scene,
			'Games Room',
			'2-games-room'
		);

		createHotspot(
			allScenes['1-cards-room'].scene,
			{ yaw: 1.7683214992527319, pitch: 0.018700769490758162 },
			allScenes['5-mens-spa'].scene,
			`Men's Spa`,
			'5-mens-spa'
		);

		createHotspot(
			allScenes['1-cards-room'].scene,
			{ yaw: 1.2937897886445082, pitch: -0.012916493926862671 },
			allScenes['8-womens-spa'].scene,
			"Women's Spa",
			'8-womens-spa'
		);

		// 2-games-room hotspots
		createHotspot(
			allScenes['2-games-room'].scene,
			{ yaw: -0.004812022893316836, pitch: -0.0448276191500927 },
			allScenes['1-cards-room'].scene,
			'Cards Room',
			'1-cards-room'
		);

		createHotspot(
			allScenes['2-games-room'].scene,
			{ yaw: -1.6017669340682659, pitch: 0.004826872859247544 },
			allScenes['5-mens-spa'].scene,
			"Men's Spa",
			'5-mens-spa'
		);

		createHotspot(
			allScenes['2-games-room'].scene,
			{ yaw: -0.8201684173757613, pitch: -0.08543697882208079 },
			allScenes['4-jacuzzi-lounge'].scene,
			'Jacuzzi Lounge',
			'4-jacuzzi-lounge'
		);

		// 3-gymnasium---weights hotspots
		createHotspot(
			allScenes['3-gymnasium---weights'].scene,
			{ yaw: -0.7070824783205438, pitch: -0.03351535078960666 },
			allScenes['6-squash-court'].scene,
			'Squash Court',
			'6-squash-court'
		);

		createHotspot(
			allScenes['3-gymnasium---weights'].scene,
			{ yaw: 0.2355561589410975, pitch: 0.0007622701808784882 },
			allScenes['7-yoga-area'].scene,
			'Yoga Area',
			'7-yoga-area'
		);

		createHotspot(
			allScenes['3-gymnasium---weights'].scene,
			{ yaw: -1.826035047376898, pitch: 0.006047984342231416 },
			allScenes['5-mens-spa'].scene,
			"Men's Spa",
			'5-mens-spa'
		);

		createHotspot(
			allScenes['3-gymnasium---weights'].scene,
			{ yaw: -2.4108177852399795, pitch: -0.024710789364066343 },
			allScenes['8-womens-spa'].scene,
			"Women's Spa",
			'8-womens-spa'
		);

		// 4-jacuzzi-lounge hotspots
		createHotspot(
			allScenes['4-jacuzzi-lounge'].scene,
			{ yaw: -1.8427462135739834, pitch: 0.002194480912674379 },
			allScenes['3-gymnasium---weights'].scene,
			'Gymnasium - Weights',
			'3-gymnasium---weights'
		);

		createHotspot(
			allScenes['4-jacuzzi-lounge'].scene,
			{ yaw: -1.5589666023453574, pitch: 0.08441562963137983 },
			allScenes['2-games-room'].scene,
			'Games Room',
			'2-games-room'
		);

		createHotspot(
			allScenes['4-jacuzzi-lounge'].scene,
			{ yaw: -0.25641461654041997, pitch: -0.08830798940178042 },
			allScenes['1-cards-room'].scene,
			'Cards Room',
			'1-cards-room'
		);

		createHotspot(
			allScenes['4-jacuzzi-lounge'].scene,
			{ yaw: 0.023783974414238074, pitch: -0.04702765892405125 },
			allScenes['2-games-room'].scene,
			'Games Room',
			'2-games-room'
		);

		// 5-mens-spa hotspots
		createHotspot(
			allScenes['5-mens-spa'].scene,
			{ yaw: 1.4904187822464605, pitch: 0.25640242754236553 },
			allScenes['8-womens-spa'].scene,
			"Women's Spa",
			'8-womens-spa'
		);

		createHotspot(
			allScenes['5-mens-spa'].scene,
			{ yaw: -2.858950870219033, pitch: -0.014160808790796509 },
			allScenes['6-squash-court'].scene,
			'Squash Court',
			'6-squash-court'
		);

		createHotspot(
			allScenes['5-mens-spa'].scene,
			{ yaw: 2.4151120670191606, pitch: -0.0013113674185465385 },
			allScenes['3-gymnasium---weights'].scene,
			'Gymnasium - Weights',
			'3-gymnasium---weights'
		);

		createHotspot(
			allScenes['5-mens-spa'].scene,
			{ yaw: -1.7450276745746631, pitch: -0.1906309998388167 },
			allScenes['4-jacuzzi-lounge'].scene,
			'Jacuzzi Lounge',
			'4-jacuzzi-lounge'
		);

		// 6-squash-court hotspots
		createHotspot(
			allScenes['6-squash-court'].scene,
			{ yaw: -0.6873591118528779, pitch: -0.03731288689007606 },
			allScenes['3-gymnasium---weights'].scene,
			'Gymnasium - Weights',
			'3-gymnasium---weights'
		);

		createHotspot(
			allScenes['6-squash-court'].scene,
			{ yaw: -1.2741912074014543, pitch: -0.056938099431013356 },
			allScenes['5-mens-spa'].scene,
			"Men's Spa",
			'5-mens-spa'
		);

		createHotspot(
			allScenes['6-squash-court'].scene,
			{ yaw: -1.6351429413357934, pitch: -0.004924353836889495 },
			allScenes['8-womens-spa'].scene,

			"Women's Spa",
			'8-womens-spa'
		);

		createHotspot(
			allScenes['6-squash-court'].scene,
			{ yaw: -0.4794097307810006, pitch: 0.0021237221170835596 },
			allScenes['4-jacuzzi-lounge'].scene,
			'Jacuzzi Lounge',
			'4-jacuzzi-lounge'
		);

		// 7-yoga-area hotspots
		createHotspot(
			allScenes['7-yoga-area'].scene,
			{ yaw: -0.6458537315125685, pitch: -0.030196089281098182 },
			allScenes['3-gymnasium---weights'].scene,
			'Gymnasium - Weights',
			'3-gymnasium---weights'
		);

		createHotspot(
			allScenes['7-yoga-area'].scene,
			{ yaw: 2.3525268831091566, pitch: -0.061817722198117764 },
			allScenes['1-cards-room'].scene,
			'Cards Room',
			'1-cards-room'
		);

		createHotspot(
			allScenes['7-yoga-area'].scene,
			{ yaw: 1.287989735401455, pitch: 0.18654531892499016 },
			allScenes['2-games-room'].scene,
			'Games Room',
			'2-games-room'
		);

		// 8-womens-spa hotspots
		createHotspot(
			allScenes['8-womens-spa'].scene,
			{ yaw: 2.744356629998652, pitch: 0.027033227159382278 },
			allScenes['5-mens-spa'].scene,
			"Men's Spa",
			'5-mens-spa'
		);

		createHotspot(
			allScenes['8-womens-spa'].scene,
			{ yaw: -2.946571239060562, pitch: -0.019434902194168657 },
			allScenes['2-games-room'].scene,
			'Games Room',
			'2-games-room'
		);

		createHotspot(
			allScenes['8-womens-spa'].scene,
			{ yaw: -0.03222174605581962, pitch: 0.0659609301696289 },
			allScenes['6-squash-court'].scene,
			'Squash Court',
			'6-squash-court'
		);

		// 0-activity-room hotspots
		createHotspot(
			allScenes['0-activity-room'].scene,
			{ yaw: -2.379318749280978, pitch: -0.06682862998684413 },
			allScenes['1-gymnasium-cardio'].scene,
			'Gymnasium Cardio',
			'1-gymnasium-cardio'
		);

		createHotspot(
			allScenes['0-activity-room'].scene,
			{ yaw: -2.6539328942177622, pitch: -0.06690026657637027 },
			allScenes['2-juice-bar'].scene,
			'Juice Bar',
			'2-juice-bar'
		);

		createHotspot(
			allScenes['0-activity-room'].scene,
			{ yaw: 1.9710094947916685, pitch: -0.06834738152408093 },
			allScenes['5-salon'].scene,
			'Salon',
			'5-salon'
		);

		// 1-gymnasium-cardio hotspots
		createHotspot(
			allScenes['1-gymnasium-cardio'].scene,
			{ yaw: 0.43613587883855764, pitch: -0.04385430995670703 },
			allScenes['0-activity-room'].scene,
			'Activity Room',
			'0-activity-room'
		);

		createHotspot(
			allScenes['1-gymnasium-cardio'].scene,
			{ yaw: -2.4216424647993318, pitch: 0.035935412497922314 },
			allScenes['2-juice-bar'].scene,
			'Juice Bar',
			'2-juice-bar'
		);

		// createHotspot(
		// 	allScenes['1-gymnasium-cardio'].scene,
		// 	{ yaw: 0.7456528955546169, pitch: 0.06886501669264966 },
		// 	allScenes['4-indoor-kids-play-area'].scene,
		// 	`Indoor Kids' Play Area`,
		// 	'4-indoor-kids-play-area'
		// );

		// 2-juice-bar hotspots
		createHotspot(
			allScenes['2-juice-bar'].scene,
			{ yaw: 1.1874585385412608, pitch: 0.019438282381548433 },
			allScenes['1-gymnasium-cardio'].scene,
			'Gymnasium Cardio',
			'1-gymnasium-cardio'
		);

		createHotspot(
			allScenes['2-juice-bar'].scene,
			{ yaw: 0.2729018483789254, pitch: -0.15440436945539204 },
			allScenes['0-activity-room'].scene,
			'Activity Room',
			'0-activity-room'
		);

		// 3-business-centre hotspots
		createHotspot(
			allScenes['3-business-centre'].scene,
			{ yaw: 0.0004240863097564329, pitch: -0.07901233664095741 },
			allScenes['5-salon'].scene,
			'Salon',
			'5-salon'
		);

		createHotspot(
			allScenes['3-business-centre'].scene,
			{ yaw: 1.6140501382530053, pitch: -0.06174563810104594 },
			allScenes['2-juice-bar'].scene,
			'Juice Bar',
			'2-juice-bar'
		);

		createHotspot(
			allScenes['3-business-centre'].scene,
			{ yaw: 1.2718118985868596, pitch: -0.051012690236177605 },
			allScenes['1-gymnasium-cardio'].scene,
			'Gymnasium Cardio',
			'1-gymnasium-cardio'
		);

		// 4-indoor-kids-play-area hotspots
		createHotspot(
			allScenes['4-indoor-kids-play-area'].scene,
			{ yaw: 0.6743240673655624, pitch: -0.0821231087822909 },
			allScenes['2-juice-bar'].scene,
			'Juice Bar',
			'2-juice-bar'
		);

		createHotspot(
			allScenes['4-indoor-kids-play-area'].scene,
			{ yaw: 0.9480961584144172, pitch: 0.02206800925668162 },
			allScenes['1-gymnasium-cardio'].scene,
			'Gymnasium Cardio',
			'1-gymnasium-cardio'
		);

		createHotspot(
			allScenes['4-indoor-kids-play-area'].scene,
			{ yaw: -0.5417735697141097, pitch: 0.10527445344299124 },
			allScenes['17-kids-play-zone'].scene,
			`Kids' Play Zone`,
			'17-kids-play-zone'
		);

		createHotspot(
			allScenes['4-indoor-kids-play-area'].scene,
			{ yaw: 0.21579402229819955, pitch: -0.0751290598540546 },
			allScenes['10-fitness-corner'].scene,
			'Fitness Corner',
			'10-fitness-corner'
		);

		// 5-salon hotspots
		createHotspot(
			allScenes['5-salon'].scene,
			{ yaw: -2.631985710751028, pitch: -0.013095431624337905 },
			allScenes['2-juice-bar'].scene,
			'Juice Bar',
			'2-juice-bar'
		);

		createHotspot(
			allScenes['5-salon'].scene,
			{ yaw: -1.2963633003907589, pitch: -0.015532041409226593 },
			allScenes['1-gymnasium-cardio'].scene,
			'Gymnasium Cardio',
			'1-gymnasium-cardio'
		);

		createHotspot(
			allScenes['5-salon'].scene,
			{ yaw: 3.094146340819888, pitch: -0.006105246193559566 },
			allScenes['17-kids-play-zone'].scene,
			`Kids' Play Zone`,
			'17-kids-play-zone'
		);

		// 6-mini-theatre hotspots
		createHotspot(
			allScenes['6-mini-theatre'].scene,
			{ yaw: 1.363322915255134, pitch: -0.08393040772180527 },
			allScenes['13-landscape-garden'].scene,
			'Landscape Garden',
			'13-landscape-garden'
		);

		// 7-aqua-deck hotspots
		createHotspot(
			allScenes['7-aqua-deck'].scene,
			{ yaw: 0.0780632200119129, pitch: -0.012173716506394427 },
			allScenes['13-landscape-garden'].scene,
			'Landscape Garden',
			'13-landscape-garden'
		);

		createHotspot(
			allScenes['7-aqua-deck'].scene,
			{ yaw: 1.143349098271372, pitch: 0.015300140511126159 },
			allScenes['9-family-pavilion'].scene,
			'Family Pavilion',
			'9-family-pavilion'
		);

		createHotspot(
			allScenes['7-aqua-deck'].scene,
			{ yaw: 0.8611024209538893, pitch: -0.037868262705833544 },
			allScenes['2-juice-bar'].scene,
			'Juice Bar',
			'2-juice-bar'
		);

		// 8-event-deck-lawn hotspots
		createHotspot(
			allScenes['8-event-deck-lawn'].scene,
			{ yaw: -1.3872495261800815, pitch: -0.016306030282921213 },
			allScenes['2-juice-bar'].scene,
			'Juice Bar',
			'2-juice-bar'
		);

		createHotspot(
			allScenes['8-event-deck-lawn'].scene,
			{ yaw: -0.48192709746882656, pitch: 0.026509078895905702 },
			allScenes['1-gymnasium-cardio'].scene,
			'Gymnasium Cardio',
			'1-gymnasium-cardio'
		);

		createHotspot(
			allScenes['8-event-deck-lawn'].scene,
			{ yaw: 0.9739647990806475, pitch: -0.002921753765155799 },
			allScenes['7-aqua-deck'].scene,
			'Aqua Deck',
			'7-aqua-deck'
		);

		createHotspot(
			allScenes['8-event-deck-lawn'].scene,
			{ yaw: 0.4759425902580645, pitch: 0.05954653338569926 },
			allScenes['15-spa-pod'].scene,
			'Spa Pod',
			'15-spa-pod'
		);

		// 9-family-pavilion hotspots
		createHotspot(
			allScenes['9-family-pavilion'].scene,
			{ yaw: -2.450261026234692, pitch: 0.245751761153727 },
			allScenes['11-kids-pool'].scene,
			`Kids' Pool`,
			'11-kids-pool'
		);

		createHotspot(
			allScenes['9-family-pavilion'].scene,
			{ yaw: -3.1220635472487075, pitch: 0.006627724282225245 },
			allScenes['12-lap-pool'].scene,
			'Lap Pool',
			'12-lap-pool'
		);

		createHotspot(
			allScenes['9-family-pavilion'].scene,
			{ yaw: -2.1912299197358287, pitch: -0.016225976544323117 },
			allScenes['7-aqua-deck'].scene,
			'Aqua Deck',
			'7-aqua-deck'
		);

		createHotspot(
			allScenes['9-family-pavilion'].scene,
			{ yaw: 1.3931122395785707, pitch: -0.03352677884085509 },
			allScenes['1-gymnasium-cardio'].scene,
			'Gymnasium Cardio',
			'1-gymnasium-cardio'
		);

		createHotspot(
			allScenes['9-family-pavilion'].scene,
			{ yaw: 0.8953623527861474, pitch: -0.006459030092884177 },
			allScenes['2-juice-bar'].scene,
			'Juice Bar',
			'2-juice-bar'
		);

		createHotspot(
			allScenes['9-family-pavilion'].scene,
			{ yaw: -0.11600923211397784, pitch: 0.08982476172847242 },
			allScenes['8-event-deck-lawn'].scene,
			'Event Deck Lawn',
			'8-event-deck-lawn'
		);

		// 10-fitness-corner hotspots
		createHotspot(
			allScenes['10-fitness-corner'].scene,
			{ yaw: -2.7507537183902464, pitch: -0.022309706003500906 },
			allScenes['17-kids-play-zone'].scene,
			`Kids' Play Zone`,
			'17-kids-play-zone'
		);

		createHotspot(
			allScenes['10-fitness-corner'].scene,
			{ yaw: 0.6757914716698359, pitch: -0.08316858144510775 },
			allScenes['2-juice-bar'].scene,
			'Juice Bar',
			'2-juice-bar'
		);

		createHotspot(
			allScenes['10-fitness-corner'].scene,
			{ yaw: 1.0697046486070683, pitch: -0.03633467419144765 },
			allScenes['8-event-deck-lawn'].scene,
			'Event Deck Lawn',
			'8-event-deck-lawn'
		);

		// 11-kids-pool hotspots
		createHotspot(
			allScenes['11-kids-pool'].scene,
			{ yaw: 1.4947830161092241, pitch: -0.04600185329191575 },
			allScenes['7-aqua-deck'].scene,
			'Aqua Deck',
			'7-aqua-deck'
		);

		createHotspot(
			allScenes['11-kids-pool'].scene,
			{ yaw: 0.02808069747572084, pitch: -0.0063046711907759345 },
			allScenes['12-lap-pool'].scene,
			'Lap Pool',
			'12-lap-pool'
		);

		createHotspot(
			allScenes['11-kids-pool'].scene,
			{ yaw: -1.7392974611180616, pitch: -0.06570971425203531 },
			allScenes['1-gymnasium-cardio'].scene,
			'Gymnasium Cardio',
			'1-gymnasium-cardio'
		);

		createHotspot(
			allScenes['11-kids-pool'].scene,
			{ yaw: -3.063570939178465, pitch: 0.002517755379638942 },
			allScenes['8-event-deck-lawn'].scene,
			'Event Deck Lawn',
			'8-event-deck-lawn'
		);

		// 12-lap-pool hotspots
		createHotspot(
			allScenes['12-lap-pool'].scene,
			{ yaw: 0.6317049432481419, pitch: 0.05832382998934271 },
			allScenes['11-kids-pool'].scene,
			`Kids' Pool`,
			'11-kids-pool'
		);

		createHotspot(
			allScenes['12-lap-pool'].scene,
			{ yaw: -0.16207270990033606, pitch: -0.023781018710707258 },
			allScenes['13-landscape-garden'].scene,
			'Landscape Garden',
			'13-landscape-garden'
		);

		createHotspot(
			allScenes['12-lap-pool'].scene,
			{ yaw: 1.040736599958013, pitch: -0.023983874028640884 },
			allScenes['1-gymnasium-cardio'].scene,
			'Gymnasium Cardio',
			'1-gymnasium-cardio'
		);

		createHotspot(
			allScenes['12-lap-pool'].scene,
			{ yaw: 0.2872702866164101, pitch: -0.048171657634984655 },
			allScenes['17-kids-play-zone'].scene,
			`Kids' Play Zone`,
			'17-kids-play-zone'
		);

		// 13-landscape-garden hotspots
		createHotspot(
			allScenes['13-landscape-garden'].scene,
			{ yaw: 0.5887643961211744, pitch: -0.06372175726589902 },
			allScenes['7-aqua-deck'].scene,
			'Aqua Deck',
			'7-aqua-deck'
		);

		createHotspot(
			allScenes['13-landscape-garden'].scene,
			{ yaw: -0.2069592211306741, pitch: -0.04637364540345601 },
			allScenes['12-lap-pool'].scene,
			'Lap Pool',
			'12-lap-pool'
		);

		createHotspot(
			allScenes['13-landscape-garden'].scene,
			{ yaw: -0.9502734899066034, pitch: 0.02975977681205677 },
			allScenes['11-kids-pool'].scene,
			`Kids' Pool`,
			'11-kids-pool'
		);

		createHotspot(
			allScenes['13-landscape-garden'].scene,
			{ yaw: -1.8845368532048834, pitch: -0.08031510810379494 },
			allScenes['8-event-deck-lawn'].scene,
			'Event Deck Lawn',
			'8-event-deck-lawn'
		);

		// 14-multipurpose-lawn hotspots
		createHotspot(
			allScenes['14-multipurpose-lawn'].scene,
			{ yaw: 2.4522899435483927, pitch: -0.04891555271526116 },
			allScenes['2-juice-bar'].scene,
			'Juice Bar',
			'2-juice-bar'
		);

		createHotspot(
			allScenes['14-multipurpose-lawn'].scene,
			{ yaw: -2.4552582490919512, pitch: 0.036064870621306255 },
			allScenes['8-event-deck-lawn'].scene,
			'Event Deck Lawn',
			'8-event-deck-lawn'
		);

		createHotspot(
			allScenes['14-multipurpose-lawn'].scene,
			{ yaw: 0.9949437327188146, pitch: -0.040394766082146205 },
			allScenes['10-fitness-corner'].scene,
			'Fitness Corner',
			'10-fitness-corner'
		);

		createHotspot(
			allScenes['14-multipurpose-lawn'].scene,
			{ yaw: 0.5427504284366247, pitch: -0.07319203767604066 },
			allScenes['4-indoor-kids-play-area'].scene,
			`Indoor Kids' Play Area`,
			'4-indoor-kids-play-area'
		);

		// 15-spa-pod hotspots
		createHotspot(
			allScenes['15-spa-pod'].scene,
			{ yaw: 0.8620907479063931, pitch: 0.017277743195446504 },
			allScenes['11-kids-pool'].scene,
			`Kids' Pool`,
			'11-kids-pool'
		);

		createHotspot(
			allScenes['15-spa-pod'].scene,
			{ yaw: 1.8983203636324637, pitch: 0.11689073821417928 },
			allScenes['12-lap-pool'].scene,
			'Lap Pool',
			'12-lap-pool'
		);

		createHotspot(
			allScenes['15-spa-pod'].scene,
			{ yaw: -0.1642052698877965, pitch: 0.056628526025718884 },
			allScenes['13-landscape-garden'].scene,
			'Landscape Garden',
			'13-landscape-garden'
		);

		createHotspot(
			allScenes['15-spa-pod'].scene,
			{ yaw: 0.3962808759888823, pitch: -0.0100446309092046 },
			allScenes['14-multipurpose-lawn'].scene,
			'Multipurpose Lawn',
			'14-multipurpose-lawn'
		);

		// 16-walking-track hotspots
		createHotspot(
			allScenes['16-walking-track'].scene,
			{ yaw: 0.54804268325206, pitch: 0.0535812335053798 },
			allScenes['13-landscape-garden'].scene,
			'Landscape Garden',
			'13-landscape-garden'
		);

		createHotspot(
			allScenes['16-walking-track'].scene,
			{ yaw: -0.9731343288685341, pitch: -0.028181665748938656 },
			allScenes['1-gymnasium-cardio'].scene,
			'Gymnasium Cardio',
			'1-gymnasium-cardio'
		);

		createHotspot(
			allScenes['16-walking-track'].scene,
			{ yaw: -1.1823590166696754, pitch: -0.03347249892532034 },
			allScenes['2-juice-bar'].scene,
			'Juice Bar',
			'2-juice-bar'
		);

		createHotspot(
			allScenes['16-walking-track'].scene,
			{ yaw: -1.6765079643505345, pitch: 0.036088320956903175 },
			allScenes['13-landscape-garden'].scene,
			'Landscape Garden',
			'13-landscape-garden'
		);

		// 17-kids-play-zone hotspots
		createHotspot(
			allScenes['17-kids-play-zone'].scene,
			{ yaw: 2.7470729920265375, pitch: 0.012736981181785012 },
			allScenes['4-indoor-kids-play-area'].scene,
			'Indoor Kids',
			'Play Area',
			'4-indoor-kids-play-area'
		);

		createHotspot(
			allScenes['17-kids-play-zone'].scene,
			{ yaw: 0.612842633509695, pitch: -0.041936008947633496 },
			allScenes['10-fitness-corner'].scene,
			'Fitness Corner',
			'10-fitness-corner'
		);

		createHotspot(
			allScenes['17-kids-play-zone'].scene,
			{ yaw: 0.9009348813850337, pitch: -0.1028363058039723 },
			allScenes['12-lap-pool'].scene,
			'Lap Pool',
			'12-lap-pool'
		);

		createHotspot(
			allScenes['17-kids-play-zone'].scene,
			{ yaw: 1.2152092759615147, pitch: -0.04167749684888378 },
			allScenes['13-landscape-garden'].scene,
			'Landscape Garden',
			'13-landscape-garden'
		);

		// 18-social-pods hotspots
		createHotspot(
			allScenes['18-social-pods'].scene,
			{ yaw: 1.9163877830095366, pitch: 0.13576255710805185 },
			allScenes['2-juice-bar'].scene,
			'Juice Bar',
			'2-juice-bar'
		);

		createHotspot(
			allScenes['18-social-pods'].scene,
			{ yaw: -2.7093616541651233, pitch: 0.018041234162105013 },
			allScenes['12-lap-pool'].scene,
			'Lap Pool',
			'12-lap-pool'
		);

		createHotspot(
			allScenes['18-social-pods'].scene,
			{ yaw: -1.4282163306509918, pitch: 0.09607550876523163 },
			allScenes['8-event-deck-lawn'].scene,
			'Event Deck Lawn',
			'8-event-deck-lawn'
		);

		createHotspot(
			allScenes['18-social-pods'].scene,
			{ yaw: 0.3122815105967014, pitch: -0.0064776667460844095 },
			allScenes['14-multipurpose-lawn'].scene,
			'Multipurpose Lawn',
			'14-multipurpose-lawn'
		);

		// 0-entrance-1 hotspots
		createHotspot(
			allScenes['0-entrance-1'].scene,
			{ yaw: 2.189883491518902, pitch: 0.0759680939265035 },
			allScenes['3-drop-off'].scene,
			'Drop-Off',
			'3-drop-off'
		);

		createHotspot(
			allScenes['0-entrance-1'].scene,
			{ yaw: 2.3617708070959242, pitch: -0.01803960574101282 },
			allScenes['4-entrance-lobby'].scene,
			'Entrance Lobby',
			'4-entrance-lobby'
		);

		createHotspot(
			allScenes['0-entrance-1'].scene,
			{ yaw: 1.677605819493908, pitch: 0.019821524828712356 },
			allScenes['7-jogging-path'].scene,
			'Jogging Path',
			'7-jogging-path'
		);

		// 1-entrance-2 hotspots
		createHotspot(
			allScenes['1-entrance-2'].scene,
			{ yaw: -1.5457199420830747, pitch: 0.04171296645843192 },
			allScenes['0-entrance-1'].scene,
			'Entrance 1',
			'0-entrance-1'
		);

		createHotspot(
			allScenes['1-entrance-2'].scene,
			{ yaw: -1.2387303745221203, pitch: 0.0522307842610914 },
			allScenes['3-drop-off'].scene,
			'Drop-Off',
			'3-drop-off'
		);

		createHotspot(
			allScenes['1-entrance-2'].scene,
			{ yaw: -0.2017393690860203, pitch: -0.013155813309445819 },
			allScenes['12-jogging-track---500m'].scene,
			'Jogging Track - 500m',
			'12-jogging-track---500m'
		);

		// 2-entrance-3 hotspots
		createHotspot(
			allScenes['2-entrance-3'].scene,
			{ yaw: -1.4584711601321168, pitch: -0.12300573447430452 },
			allScenes['12-jogging-track---500m'].scene,
			'Jogging Track - 500m',
			'12-jogging-track---500m'
		);

		createHotspot(
			allScenes['2-entrance-3'].scene,
			{ yaw: -2.6304577455373064, pitch: -0.07110801875638018 },
			allScenes['15-pet-run'].scene,
			'Pet Run',
			'15-pet-run'
		);

		// 3-drop-off hotspots
		createHotspot(
			allScenes['3-drop-off'].scene,
			{ yaw: -2.4920476386010186, pitch: 0.019429413444967736 },
			allScenes['7-jogging-path'].scene,
			'Jogging Path',
			'7-jogging-path'
		);

		createHotspot(
			allScenes['3-drop-off'].scene,
			{ yaw: 2.596957849397782, pitch: 0.015058059682701241 },
			allScenes['0-entrance-1'].scene,
			'Entrance 1',
			'0-entrance-1'
		);

		createHotspot(
			allScenes['3-drop-off'].scene,
			{ yaw: -0.3272831796615616, pitch: -0.11927984162118932 },
			allScenes['4-entrance-lobby'].scene,
			'Entrance Lobby',
			'4-entrance-lobby'
		);

		// 4-entrance-lobby hotspots
		createHotspot(
			allScenes['4-entrance-lobby'].scene,
			{ yaw: -0.692806089370734, pitch: 0.030198328211842096 },
			allScenes['0-entrance-1'].scene,
			'Entrance 1',
			'0-entrance-1'
		);

		createHotspot(
			allScenes['4-entrance-lobby'].scene,
			{ yaw: -1.0738710517486894, pitch: 0.04714986325645398 },
			allScenes['3-drop-off'].scene,
			'Drop-Off',
			'3-drop-off'
		);

		createHotspot(
			allScenes['4-entrance-lobby'].scene,
			{ yaw: -2.1882874858455192, pitch: -0.01954698354456852 },
			allScenes['5-lift-lobby'].scene,
			'Lift Lobby',
			'5-lift-lobby'
		);

		// 5-lift-lobby hotspots
		createHotspot(
			allScenes['5-lift-lobby'].scene,
			{ yaw: -0.109765726508261, pitch: -0.06316050592938893 },
			allScenes['4-entrance-lobby'].scene,
			'Entrance Lobby',
			'4-entrance-lobby'
		);

		createHotspot(
			allScenes['5-lift-lobby'].scene,
			{ yaw: -0.3307476272600134, pitch: -0.20703472248541388 },
			allScenes['0-entrance-1'].scene,
			'Entrance 1',
			'0-entrance-1'
		);

		createHotspot(
			allScenes['5-lift-lobby'].scene,
			{ yaw: -2.542798687719145, pitch: -0.06996981960002735 },
			allScenes['15-pet-run'].scene,
			'Pet Run',
			'15-pet-run'
		);

		createHotspot(
			allScenes['5-lift-lobby'].scene,
			{ yaw: 0.86761019887415, pitch: -0.014736286099340745 },
			allScenes['6-convinience-store'].scene,
			'Convinience Store',
			'6-convinience-store'
		);

		// 6-convinience-store hotspots
		createHotspot(
			allScenes['6-convinience-store'].scene,
			{ yaw: -2.0232738699536768, pitch: -0.002251326477477278 },
			allScenes['4-entrance-lobby'].scene,
			'Entrance Lobby',
			'4-entrance-lobby'
		);

		createHotspot(
			allScenes['6-convinience-store'].scene,
			{ yaw: -0.5390835598891481, pitch: -0.0264600536974271 },
			allScenes['15-pet-run'].scene,
			'Pet Run',
			'15-pet-run'
		);

		// 7-jogging-path hotspots
		createHotspot(
			allScenes['7-jogging-path'].scene,
			{ yaw: -1.6536860928479502, pitch: -0.040092429726609424 },
			allScenes['0-entrance-1'].scene,
			'Entrance 1',
			'0-entrance-1'
		);

		createHotspot(
			allScenes['7-jogging-path'].scene,
			{ yaw: -2.8715679632823523, pitch: -0.06658417060047839 },
			allScenes['3-drop-off'].scene,
			'Drop-Off',
			'3-drop-off'
		);

		createHotspot(
			allScenes['7-jogging-path'].scene,
			{ yaw: 2.4830981304145743, pitch: -0.21580258989697398 },
			allScenes['3-drop-off'].scene,
			'Drop-Off',
			'3-drop-off'
		);

		createHotspot(
			allScenes['7-jogging-path'].scene,
			{ yaw: -2.3728078802943777, pitch: -0.03741681750248027 },
			allScenes['12-jogging-track---500m'].scene,
			'Jogging Track - 500m',
			'12-jogging-track---500m'
		);

		// 8-garden-pavillion hotspots
		createHotspot(
			allScenes['8-garden-pavillion'].scene,
			{ yaw: -1.6469666332543316, pitch: 0.039209786515572276 },
			allScenes['7-jogging-path'].scene,
			'Jogging Path',
			'7-jogging-path'
		);

		createHotspot(
			allScenes['8-garden-pavillion'].scene,
			{ yaw: -1.2452720655214016, pitch: -0.05143204105093879 },
			allScenes['0-entrance-1'].scene,
			'Entrance 1',
			'0-entrance-1'
		);

		createHotspot(
			allScenes['8-garden-pavillion'].scene,
			{ yaw: 0.6809190596472305, pitch: 0.0014778524775636015 },
			allScenes['12-jogging-track---500m'].scene,
			'Jogging Track - 500m',
			'12-jogging-track---500m'
		);

		createHotspot(
			allScenes['8-garden-pavillion'].scene,
			{ yaw: -0.7261601655166992, pitch: -0.1288032160413657 },
			allScenes['9-kids-play-area'].scene,
			`Kids' Play Area`,
			'9-kids-play-area'
		);

		createHotspot(
			allScenes['8-garden-pavillion'].scene,
			{ yaw: -2.1727776902057823, pitch: -0.09013329623888922 },
			allScenes['10-box-cricket'].scene,
			'Box Cricket',
			'10-box-cricket'
		);

		// 9-kids-play-area hotspots
		createHotspot(
			allScenes['9-kids-play-area'].scene,
			{ yaw: 1.8992829581530408, pitch: -0.021908594480994026 },
			allScenes['8-garden-pavillion'].scene,
			'Garden Pavillion',
			'8-garden-pavillion'
		);

		createHotspot(
			allScenes['9-kids-play-area'].scene,
			{ yaw: -1.0028220569291904, pitch: -0.12090302294461353 },
			allScenes['0-entrance-1'].scene,
			'Entrance 1',
			'0-entrance-1'
		);

		createHotspot(
			allScenes['9-kids-play-area'].scene,
			{ yaw: -1.3701876950901504, pitch: 0.02061446914644094 },
			allScenes['7-jogging-path'].scene,
			'Jogging Path',
			'7-jogging-path'
		);

		createHotspot(
			allScenes['9-kids-play-area'].scene,
			{ yaw: 0.9449994536504711, pitch: -0.1102947315909546 },
			allScenes['12-jogging-track---500m'].scene,
			'Jogging Track - 500m',
			'12-jogging-track---500m'
		);

		createHotspot(
			allScenes['9-kids-play-area'].scene,
			{ yaw: -2.5799346811841986, pitch: 0.06923249406495735 },
			allScenes['10-box-cricket'].scene,
			'Box Cricket',
			'10-box-cricket'
		);

		// 10-box-cricket hotspots
		createHotspot(
			allScenes['10-box-cricket'].scene,
			{ yaw: 0.23841312470420561, pitch: -0.0021273034142730296 },
			allScenes['9-kids-play-area'].scene,
			`Kids' Play Area`,
			'9-kids-play-area'
		);

		createHotspot(
			allScenes['10-box-cricket'].scene,
			{ yaw: 1.0555690416149979, pitch: -0.015486646270765902 },
			allScenes['8-garden-pavillion'].scene,
			'Garden Pavillion',
			'8-garden-pavillion'
		);

		createHotspot(
			allScenes['10-box-cricket'].scene,
			{ yaw: -1.1672726307034598, pitch: -0.07016522100382616 },
			allScenes['0-entrance-1'].scene,
			'Entrance 1',
			'0-entrance-1'
		);

		createHotspot(
			allScenes['10-box-cricket'].scene,
			{ yaw: -0.864729161543373, pitch: 0.06053363140842549 },
			allScenes['7-jogging-path'].scene,
			'Jogging Path',
			'7-jogging-path'
		);

		// 11-social-pods hotspots
		createHotspot(
			allScenes['11-social-pods'].scene,
			{ yaw: 0.11233830645791443, pitch: -0.005532561550955606 },
			allScenes['12-jogging-track---500m'].scene,
			'Jogging Track - 500m',
			'12-jogging-track---500m'
		);

		createHotspot(
			allScenes['11-social-pods'].scene,
			{ yaw: -2.7628126975827207, pitch: 0.013300909398088478 },
			allScenes['7-jogging-path'].scene,
			'Jogging Path',
			'7-jogging-path'
		);

		createHotspot(
			allScenes['11-social-pods'].scene,
			{ yaw: -3.019559245737275, pitch: -0.015138956343104581 },
			allScenes['0-entrance-1'].scene,
			'Entrance 1',
			'0-entrance-1'
		);

		// 12-jogging-track---500m hotspots
		createHotspot(
			allScenes['12-jogging-track---500m'].scene,
			{ yaw: -1.6571488769957554, pitch: 0.4645368155388123 },
			allScenes['13-banquet-drop-off'].scene,
			'Banquet Drop Off',
			'13-banquet-drop-off'
		);

		createHotspot(
			allScenes['12-jogging-track---500m'].scene,
			{ yaw: -2.441797249016668, pitch: -0.092299605580763 },
			allScenes['0-entrance-1'].scene,
			'Entrance 1',
			'0-entrance-1'
		);

		createHotspot(
			allScenes['12-jogging-track---500m'].scene,
			{ yaw: -2.1694664693588965, pitch: 0.01928868840515996 },
			allScenes['7-jogging-path'].scene,
			'Jogging Path',
			'7-jogging-path'
		);

		createHotspot(
			allScenes['12-jogging-track---500m'].scene,
			{ yaw: -1.602780748945218, pitch: 0.006311970228750852 },
			allScenes['14-banquet-hall'].scene,
			'Banquet Hall',
			'14-banquet-hall'
		);

		// 13-banquet-drop-off hotspots
		createHotspot(
			allScenes['13-banquet-drop-off'].scene,
			{ yaw: -2.4495504640518604, pitch: -0.04670821814150017 },
			allScenes['14-banquet-hall'].scene,
			'Banquet Hall',
			'14-banquet-hall'
		);

		createHotspot(
			allScenes['13-banquet-drop-off'].scene,
			{ yaw: 0.5735703228318876, pitch: 0.0897248311566834 },
			allScenes['13-banquet-drop-off'].scene,
			'Banquet Drop Off',
			'13-banquet-drop-off'
		);

		createHotspot(
			allScenes['13-banquet-drop-off'].scene,
			{ yaw: 2.2525143321531598, pitch: -0.057885220580986996 },
			allScenes['0-entrance-1'].scene,
			'Entrance 1',
			'0-entrance-1'
		);

		// 14-banquet-hall hotspots
		createHotspot(
			allScenes['14-banquet-hall'].scene,
			{ yaw: 0.49304429492998736, pitch: -0.01686833614342298 },
			allScenes['13-banquet-drop-off'].scene,
			'Banquet Drop Off',
			'13-banquet-drop-off'
		);

		createHotspot(
			allScenes['14-banquet-hall'].scene,
			{ yaw: -0.766941162036261, pitch: 0.02565875940629958 },
			allScenes['12-jogging-track---500m'].scene,
			'Jogging Track - 500m',
			'12-jogging-track---500m'
		);

		createHotspot(
			allScenes['14-banquet-hall'].scene,
			{ yaw: 1.9011386992060757, pitch: -0.0355408717102339 },
			allScenes['0-entrance-1'].scene,
			'Entrance 1',
			'0-entrance-1'
		);

		// 15-pet-run hotspots
		createHotspot(
			allScenes['15-pet-run'].scene,
			{ yaw: -0.874671130160781, pitch: -0.05778698951052519 },
			allScenes['0-entrance-1'].scene,
			'Entrance 1',
			'0-entrance-1'
		);

		createHotspot(
			allScenes['15-pet-run'].scene,
			{ yaw: -1.126389581763057, pitch: 0.09364670250934459 },
			allScenes['7-jogging-path'].scene,
			'Jogging Path',
			'7-jogging-path'
		);

		createHotspot(
			allScenes['15-pet-run'].scene,
			{ yaw: -3.10242555094759, pitch: -0.04210035524025457 },
			allScenes['12-jogging-track---500m'].scene,
			'Jogging Track - 500m',
			'12-jogging-track---500m'
		);

		var pano = document.getElementById('pano');

		pano.addEventListener('click', function (e) {
			var imgHotspot = document.createElement('img');
			imgHotspot.src = '';
			imgHotspot.classList.add('hotspot');
			imgHotspot.addEventListener('click', function () {
				alert('CLICK!');
			});
			// // http://www.marzipano.net/reference/Viewer.html#view dapat dari view di index.js
			var view = viewer.view();
			var loc = view.screenToCoordinates({ x: e.clientX, y: e.clientY });
			var position = { yaw: loc._yaw, pitch: loc._pitch };
			// http://www.marzipano.net/reference/Viewer.html#scene dapat dari scene
			console.log(view.screenToCoordinates({ x: e.clientX, y: e.clientY }));
			window.hotspotData = JSON.stringify(view.screenToCoordinates({ x: e.clientX, y: e.clientY }));
			console.log(view._yaw + ' - ' + view._pitch);
			var sceness = viewer.scene();
			sceness.hotspotContainer().createHotspot(imgHotspot, position);
		});

		unsubscribeHotSpot = hotspotName.subscribe((changedHotspot) => {
			console.log('$currentUI.hotspot', changedHotspot);
			const ignoredHotspots = [
				'exterior-a',
				'amenities',
				'kalpataru-one',
				'podium-view',
				'elevation',
				'exterior',
				'day',
				'night'
			];
			if (ignoredHotspots.includes(changedHotspot)) {
				return;
			}
			if (changedHotspot != 'Exterior' && changedHotspot != 'ExteriorImg') {
				if (!$page.url.href.includes('area=highlights')) {
					$ishighlights = false;
				}
				console.log(
					'current hotspot looker',
					changedHotspot,
					$page.url.href.includes('area=highlights')
				);
				allScenes[changedHotspot].scene.switchTo();
				window.view = allScenes[changedHotspot].view;
				window.dragControlMethod.addEventListener('parameterDynamics', function () {
					document.querySelectorAll('.info-hotspot').forEach((e) => {
						setTimeout(() => {
							e.style.transform = resetNegativeTranslations(
								e.style.transform,
								e.querySelector('.hotspot')
							);
						}, 500);
					});
				});
			}
		});

		setTimeout(() => {
			window?.marzipano?.next();
		}, 4000);

		window.addEventListener('keydown', (event) => {
			if (event.key === '?' || event.key === '?') {
				const hotspot = $hotspotName;

				const finalTextToCopy = `from hotspot: ${hotspot} \n  \nPosition: ${window?.hotspotData || ''}`;
				navigator.clipboard.writeText(finalTextToCopy).then(
					() => {
						alert(`Copied hotspot info for  ":\n${finalTextToCopy}`);
					},
					() => {
						alert('Failed to copy hotspot info');
					}
				);
			}
		});
	});
	onDestroy(() => {
		if (unsubscribeHotSpot) unsubscribeHotSpot();
	});
</script>

{#if !$ishighlights}
	<div class="left-panel-wrapper">
		<div class="left-panel p-2">
			<div class="left-panel--header flex justify-between gap-[2rem] lg:gap-[5rem]">
				<div class="left-title flex items-center font-bold">
					<svg
						class="mr-2"
						width="25"
						height="25"
						viewBox="0 0 32 32"
						fill="none"
						xmlns="http://www.w3.org/2000/svg"
					>
						<path
							d="M8.90798 13.3021C8.90798 10.1749 8.89311 7.0624 8.90798 3.93517C8.92285 2.07654 10.2613 0.527682 12.031 0.262163C13.9196 -0.0328582 15.6596 0.984965 16.2395 2.74034C16.3436 3.06486 16.418 3.40414 16.4477 3.74341C16.5072 4.46621 16.0462 5.02675 15.377 5.11526C14.7375 5.18901 14.2022 4.73173 14.0832 3.99418C13.9345 3.03536 13.414 2.53382 12.5961 2.57808C11.7633 2.62233 11.2874 3.21237 11.2874 4.21544C11.2874 4.76123 11.2725 5.29227 11.2874 5.83806C11.3171 6.47236 11.0197 7.28366 11.4064 7.68194C11.8079 8.09497 12.6258 7.8147 13.2653 7.8147C15.9124 7.82945 18.5743 7.8147 21.2214 7.82945C21.6973 7.82945 21.8758 7.72619 21.8609 7.22466C21.8312 6.11833 21.8312 5.012 21.8609 3.90567C21.9055 1.88478 23.4373 0.321167 25.4598 0.203159C27.4228 0.0999013 29.1033 1.4865 29.3858 3.47789C29.5048 4.33345 29.1033 4.9825 28.3894 5.10051C27.8392 5.18901 27.051 4.70223 27.0064 3.87617C26.9618 3.00586 26.3223 2.50432 25.5341 2.56333C24.7311 2.62233 24.2403 3.21237 24.2403 4.15644C24.2403 10.2781 24.2403 16.3998 24.2403 22.5067C24.2403 22.728 24.2552 22.9493 24.2255 23.1705C24.166 23.8196 23.6901 24.2769 23.0655 24.2916C22.4558 24.3064 21.9501 23.8491 21.8609 23.2148C21.846 23.0968 21.846 22.964 21.846 22.846C21.8312 21.6364 21.8312 21.6364 20.582 21.6364C17.7564 21.6364 14.9309 21.6512 12.1053 21.6217C11.4807 21.6217 11.1684 21.7249 11.2874 22.4182C11.332 22.6543 11.3023 22.905 11.2874 23.1558C11.2428 23.8048 10.7669 24.2769 10.1423 24.2916C9.50283 24.3064 8.96746 23.8343 8.92285 23.1558C8.89311 22.5658 8.90798 21.9757 8.90798 21.3857C8.90798 18.7157 8.90798 16.0015 8.90798 13.3021ZM16.6113 13.8184C18.1431 13.8184 19.6748 13.8036 21.2214 13.8331C21.7568 13.8479 21.9055 13.6709 21.8758 13.1693C21.8312 12.358 21.8609 11.5467 21.8758 10.7354C21.8907 10.3814 21.8014 10.2044 21.385 10.2044C18.1877 10.2191 15.0052 10.2191 11.8079 10.2044C11.4064 10.2044 11.3023 10.3519 11.3023 10.7207C11.3171 11.5615 11.332 12.3875 11.3023 13.2283C11.2725 13.7299 11.4807 13.8479 11.9417 13.8331C13.4883 13.8036 15.0498 13.8184 16.6113 13.8184ZM16.6113 19.3058C18.1728 19.3058 19.7343 19.291 21.2958 19.3205C21.7568 19.3353 21.8907 19.1878 21.8758 18.7452C21.846 18.0519 21.8609 17.3734 21.8758 16.6801C21.8758 16.3556 21.8163 16.1933 21.4296 16.1933C18.2174 16.208 14.9904 16.208 11.7781 16.1933C11.4212 16.1933 11.3023 16.3113 11.3171 16.6653C11.332 17.3586 11.3617 18.0372 11.3171 18.7305C11.2874 19.2468 11.4956 19.3353 11.9566 19.3353C13.4883 19.291 15.0498 19.3058 16.6113 19.3058Z"
							fill="#A47764"
						/>
						<path
							d="M25.8935 31.0057C24.6808 31.0057 23.661 30.5336 22.8204 29.6043C22.4483 29.206 22.1038 28.7635 21.5664 28.6013C20.6568 28.3357 19.954 28.7193 19.3615 29.4421C18.5208 30.4451 17.4873 30.9909 16.2195 31.0057C14.9792 31.0204 13.9594 30.5041 13.0913 29.5748C11.7683 28.144 10.9001 28.144 9.59098 29.5601C7.77193 31.522 5.20873 31.5367 3.40347 29.5748C2.87981 28.9995 2.32858 28.5423 1.5293 28.4832C0.867833 28.439 0.481975 27.849 0.550878 27.2147C0.619782 26.5656 1.08832 26.1378 1.74979 26.1673C2.93493 26.2116 3.92714 26.7131 4.74019 27.6129C5.15361 28.0702 5.55325 28.5275 6.21472 28.616C7.09668 28.734 7.68925 28.2472 8.24048 27.6129C9.27402 26.4329 10.5556 25.9756 12.0439 26.2263C12.981 26.3886 13.7665 26.8754 14.4142 27.5982C14.8414 28.0702 15.2686 28.5423 15.9576 28.616C16.8258 28.7045 17.377 28.2177 17.9145 27.6129C19.1685 26.2263 20.6706 25.8281 22.3794 26.3886C23.096 26.6246 23.661 27.1114 24.1847 27.6867C25.3836 28.9995 26.3345 28.9995 27.5334 27.6719C28.3464 26.7574 29.3249 26.2263 30.4962 26.1526C31.2128 26.1083 31.7089 26.5214 31.7778 27.1852C31.8467 27.8785 31.4609 28.4242 30.758 28.4832C30.0139 28.5423 29.4764 28.9405 28.9941 29.5011C28.1673 30.4894 27.1337 30.9909 25.8935 31.0057Z"
							fill="#A47764"
						/>
					</svg>
					Amenities
				</div>
				<button
					on:click={() => {
						$isAmenitiesMinimized = !$isAmenitiesMinimized;
					}}
					class="ghost-btn close-btn !px-0 !py-0"
					id="minimize-toggle-vicinity"
				>
					{#if !$isAmenitiesMinimized}
						<img src={minimizeBtn} alt="minimize" />
					{/if}
					{#if $isAmenitiesMinimized}
						<img src={maximizeBtn} alt="maximize" />
					{/if}
				</button>
			</div>

			<div class={!$isAmenitiesMinimized ? 'block' : 'hidden'}>
				<div class="pt-3">
					<div class="inner-btn-group">
						<Accordion.Root class="w-full sm:max-w-full">
							<Accordion.Item value="item-12" class="">
								<Accordion.Trigger id="grolevel-fx">Ground Floor</Accordion.Trigger>
								<Accordion.Content>
									{#each [{ id: '0-entrance-1', label: 'Entrance 1' }, { id: '1-entrance-2', label: 'Entrance 2' }, { id: '2-entrance-3', label: 'Entrance 3' }, { id: '3-drop-off', label: 'Drop - Off' }, { id: '4-entrance-lobby', label: 'Entrance Lobby' }, { id: '5-lift-lobby', label: 'Lift Lobby' }, { id: '6-convinience-store', label: 'Convenience Store' }, { id: '7-jogging-path', label: 'Jogging Path' }, { id: '12-jogging-track---500m', label: 'Jogging Path - 500m' }, { id: '8-garden-pavillion', label: 'Garden Pavilion' }, { id: '9-kids-play-area', label: "Kids' Play Area" }, { id: '10-box-cricket', label: 'Box Cricket' }, { id: '11-social-pods', label: 'Social Pods' }, { id: '15-pet-run', label: 'Pet Run' }, { id: '13-banquet-drop-off', label: 'Banquet Drop Off' }, { id: '14-banquet-hall', label: 'Banquet Hall' }] as scene}
										<button
											class={$hotspotName == scene.id
												? 'active inner-modal-btn '
												: 'inner-modal-btn '}
											id={'am-' + scene.id}
											on:click={() => ($hotspotName = scene.id)}
										>
											{scene.label}
										</button>
									{/each}
								</Accordion.Content>
							</Accordion.Item>
							<Accordion.Item value="item-13" class="">
								<Accordion.Trigger id="podium">Podium (P4 & P6)</Accordion.Trigger>
								<Accordion.Content>
									{#each [{ id: '6-squash-court', label: 'Squash Court' }, { id: '2-games-room', label: 'Games Room' }, { id: '8-womens-spa', label: "Women's Spa" }, { id: '5-mens-spa', label: "Men's Spa" }, { id: '4-jacuzzi-lounge', label: 'Jacuzzi Lounge' }, { id: '7-yoga-area', label: 'Yoga Area' }, { id: '1-cards-room', label: 'Cards Room' }, { id: '3-gymnasium---weights', label: 'Gymnasium - Weights' }] as scene}
										<button
											class={$hotspotName == scene.id
												? 'active inner-modal-btn '
												: 'inner-modal-btn '}
											id={'am-' + scene.id}
											on:click={() => ($hotspotName = scene.id)}
										>
											{scene.label}
										</button>
									{/each}
								</Accordion.Content>
							</Accordion.Item>
							<Accordion.Item value="item-14" class=" ">
								<Accordion.Trigger id="clubhouse">Clubhouse</Accordion.Trigger>
								<Accordion.Content>
									{#each [{ id: '2-juice-bar', label: 'Juice Bar' }, { id: '1-gymnasium-cardio', label: 'Gymnasium - Cardio' }, { id: '0-activity-room', label: 'Activity Room' }] as scene}
										<button
											class={$hotspotName == scene.id
												? 'active inner-modal-btn '
												: 'inner-modal-btn '}
											id={'am-' + scene.id}
											on:click={() => ($hotspotName = scene.id)}
										>
											<span class="max-w-[10rem] truncate" title={scene.label}> {scene.label}</span>
										</button>
									{/each}
								</Accordion.Content>
							</Accordion.Item>

							<Accordion.Item value="item-15" class=" ">
								<Accordion.Trigger id="eco-level">Eco Deck Level</Accordion.Trigger>
								<Accordion.Content>
									{#each [{ id: '12-lap-pool', label: 'Lap Pool' }, { id: '7-aqua-deck', label: 'Aqua Deck' }, { id: '15-spa-pod', label: 'Spa Pods' }, { id: '11-kids-pool', label: "Kids' Pool" }, { id: '9-family-pavilion', label: 'Family Pavilion' }, { id: '13-landscape-garden', label: 'Landscape Gardens' }, { id: '16-walking-track', label: 'Walking Track' }, { id: '8-event-deck-lawn', label: 'Event Deck & Lawn' }, { id: '14-multipurpose-lawn', label: 'Multipurpose Lawn' }, { id: '18-social-pods', label: 'Social Pods' }, { id: '10-fitness-corner', label: 'Fitness Corner' }, { id: '17-kids-play-zone', label: "Kids' Play Zone" }] as scene}
										<button
											class={$hotspotName == scene.id
												? 'active inner-modal-btn '
												: 'inner-modal-btn '}
											id={'am-' + scene.id}
											on:click={() => ($hotspotName = scene.id)}
										>
											<span class="max-w-[10rem] truncate" title={scene.label}> {scene.label}</span>
										</button>
									{/each}
								</Accordion.Content>
							</Accordion.Item>

							<Accordion.Item value="item-16" class=" ">
								<Accordion.Trigger id="podium-stilt">Podium Stilt</Accordion.Trigger>
								<Accordion.Content>
									{#each [{ id: '4-indoor-kids-play-area', label: "Indoor Kids' Play Area" }, { id: '5-salon', label: 'Salon' }, { id: '6-mini-theatre', label: 'Mini Theatre' }, { id: '3-business-centre', label: 'Business Centre' }] as scene}
										<button
											class={$hotspotName == scene.id
												? 'active inner-modal-btn '
												: 'inner-modal-btn '}
											id={'am-' + scene.id}
											on:click={() => ($hotspotName = scene.id)}
										>
											<span class="max-w-[10rem] truncate" title={scene.label}> {scene.label}</span>
										</button>
									{/each}
								</Accordion.Content>
							</Accordion.Item>

							<Accordion.Item value="item-17" class=" ">
								<Accordion.Trigger id="sky-level">Sky Level</Accordion.Trigger>
								<Accordion.Content>
									{#each [{ id: '4-chillout-lawn-tower-a', label: 'Chill Out Lawn Tower A' }, { id: '3-lap-pool-tower-a', label: 'Lap Pool Tower A' }, { id: '0-cozy-lounge-deck-tower-a', label: 'Cozy Lounge Deck Tower A' }, { id: '1-sky-lounge-room-1-tower-a', label: 'Sky Lounge Room 1 Tower A' }, { id: '2-sky-lounge-room-2-tower-a', label: 'Sky Lounge Room 2 Tower A' }, { id: '5-seating-planters-tower-b', label: 'Seating Planters Tower B' }, { id: '6-chillout-deck-tower-b', label: 'Chill Out Deck Tower B' }, { id: '7-lap-pool-tower-b', label: 'Lap Pool Tower B' }, { id: '8-social-deck-tower-b', label: 'Social Deck Tower B' }, { id: '12-lookout-deck-tower-c', label: 'Lookout Deck Tower C' }, { id: '11-pool-loungers-tower-c', label: 'Pool Loungers Tower C' }, { id: '9-lap-pool-tower-c', label: 'Lap Pool Tower C' }, { id: '10-chillout-deck-tower-c', label: 'Chill Out Deck Tower C' }] as scene}
										<button
											class={$hotspotName == scene.id
												? 'active inner-modal-btn '
												: 'inner-modal-btn '}
											id={'am-' + scene.id}
											on:click={() => ($hotspotName = scene.id)}
										>
											<span class="max-w-[10rem] truncate" title={scene.label}> {scene.label}</span>
										</button>
									{/each}
								</Accordion.Content>
							</Accordion.Item>
						</Accordion.Root>
					</div>
				</div>
			</div>
		</div>
	</div>
{/if}

<div id="pano"></div>
