<script>
	import bowser from 'bowser';
	import dollhouse from '$lib/images/4bhk.webp';
	import dollhouse1 from '$lib/images/5bhk.webp';
	import { onMount } from 'svelte';
	let Marzipano;
	import * as Accordion from '$lib/components/ui/accordion/index.ts';

	import minimizeBtn from '$lib/images/minimize-icon.svg';
	import maximizeBtn from '$lib/images/maximize-icon.svg';
	import { fade } from 'svelte/transition';
	import startTourImg from '$lib/images/startTourImg.svg';
	import { writable } from 'svelte/store';
	import { onDestroy } from 'svelte';
	import { setContext, getContext } from 'svelte';
	const isInteriorUnitDataMinimized = writable();
	setContext('isInteriorUnitDataMinimized', isInteriorUnitDataMinimized);

	$: isInteriorUnitDataMinimized.set(true);
	let hotspotName = getContext('hotspotName');
	let currentUI = getContext('currentUI');
	$currentUI['interiors'] = true;
	const currentbhk = writable();
	currentbhk.set('4bhk');
	let unsubscribeHotSpot;
	const startTour = writable(false);
	$: $startTour;
	const isInteriorUnitMinimized = writable();
	setContext('isInteriorUnitMinimized', isInteriorUnitMinimized);
	$: isInteriorUnitMinimized.set(true);
	const isbhklist = writable();
	setContext('isbhklist', isbhklist);
	$: isbhklist.set(true);
	// const currentbhk = writable();
	onMount(async () => {
		// @ts-expect-error because my dead marzipano doesn't have type bindings like i even cared in the first place
		Marzipano = await import('marzipano');
		let allScenes = {};
		let counter = 0;
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
		console.log('Marzipano initialized on main app', window?.dragControlMethod);
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
			var urlPrefix = 'https://assets.vestate.io/kalpataru-one/interior';

			let source = Marzipano.ImageUrlSource.fromString(`${urlPrefix}/${name}/{z}/{f}/{y}/{x}.jpg`, {
				cubeMapPreviewUrl: `${urlPrefix}/${name}/preview.jpg`
			});

			var geometry = new Marzipano.CubeGeometry(size);
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

		// Scene: 0-foyer
		create360scene(
			'0-foyer',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 },
				{ tileSize: 512, size: 4096 }
			],
			{ yaw: 3.138364710358612, pitch: 0.012098492225751656, fov: 1.4774772949317303 }
		);

		// Scene: 1-livingroom
		create360scene(
			'1-livingroom',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 },
				{ tileSize: 512, size: 4096 }
			],
			{ yaw: -3.126521848277445, pitch: -3.552713678800501e-15, fov: 1.4774772949317303 }
		);

		// Scene: 2-deck
		create360scene(
			'2-deck',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 },
				{ tileSize: 512, size: 4096 }
			],
			{ yaw: -0.000925692799508937, pitch: 0.005094749292867462, fov: 1.4774772949317303 }
		);

		// Scene: 3-dinning
		create360scene(
			'3-dinning',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 },
				{ tileSize: 512, size: 4096 }
			],
			{ yaw: 1.5763507906506256, pitch: -0.001390014559587982, fov: 1.4774772949317303 }
		);

		// Scene: 4-kitchen4bhk
		create360scene(
			'4-kitchen4bhk',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 1.9557712733596961, pitch: 0.09193567351740839, fov: 1.4774772949317303 }
		);

		// Scene: 5-utility4bhk
		create360scene(
			'5-utility4bhk',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 },
				{ tileSize: 512, size: 4096 }
			],
			{ yaw: 0.3921221541577573, pitch: -0.001104270286305109, fov: 1.4774772949317303 }
		);

		// Scene: 7-masterbed4bhkph
		create360scene(
			'7-masterbed4bhkph',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 0.019694186928044033, pitch: 0.03566324505007401, fov: 1.4774772949317303 }
		);

		// Scene: 6-family
		create360scene(
			'6-family',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 },
				{ tileSize: 512, size: 4096 }
			],
			{ yaw: 3.136938701103613, pitch: -1.7763568394002505e-15, fov: 1.4774772949317303 }
		);

		// Scene: 8-masterwardrobe
		create360scene(
			'8-masterwardrobe',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: -1.5642761743364861, pitch: 0.006496473328327923, fov: 1.4774772949317303 }
		);

		// Scene: 9-masterwashroom
		create360scene(
			'9-masterwashroom',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 },
				{ tileSize: 512, size: 4096 }
			],
			{ yaw: -1.5718878838291488, pitch: 0.013153689010604097, fov: 1.4774772949317303 }
		);

		// Scene: 10-guestroom014bhk
		create360scene(
			'10-guestroom014bhk',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 },
				{ tileSize: 512, size: 4096 }
			],
			{ yaw: -2.784281185457969, pitch: -3.552713678800501e-15, fov: 1.4774772949317303 }
		);

		// Scene: 11-guestwardrobe1
		create360scene(
			'11-guestwardrobe1',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 1.56499744087985, pitch: 0.002547374646434619, fov: 1.4774772949317303 }
		);

		// Scene: 12-guestwashroom1
		create360scene(
			'12-guestwashroom1',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 },
				{ tileSize: 512, size: 4096 }
			],
			{ yaw: 1.5749651412137275, pitch: 0.0028871248317923204, fov: 1.3809931972994007 }
		);

		// Scene: 13-kidroom
		create360scene(
			'13-kidroom',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 },
				{ tileSize: 512, size: 4096 }
			],
			{ yaw: 3.136262033162307, pitch: 0.003509262885456721, fov: 1.4774772949317303 }
		);

		// Scene: 14-kidswashroom
		create360scene(
			'14-kidswashroom',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 },
				{ tileSize: 512, size: 4096 }
			],
			{ yaw: -0.0055048464342775105, pitch: 0.008035605962465553, fov: 1.4774772949317303 }
		);

		// Scene: 15-guestroom2
		create360scene(
			'15-guestroom2',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 3.134277339311774, pitch: 0.017382085822722715, fov: 1.4774772949317303 }
		);

		// Scene: 16-guestwardrobe2
		create360scene(
			'16-guestwardrobe2',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 },
				{ tileSize: 512, size: 4096 }
			],
			{ yaw: -0.7413808750650848, pitch: 0.0006786073862343756, fov: 1.4774772949317303 }
		);

		// Scene: 17-guestwashroom2
		create360scene(
			'17-guestwashroom2',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 },
				{ tileSize: 512, size: 4096 }
			],
			{ yaw: 0.0010705797979131404, pitch: 0.002107314015320938, fov: 1.4774772949317303 }
		);

		// Scene: 18-commonwashroom
		create360scene(
			'18-commonwashroom',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 },
				{ tileSize: 512, size: 4096 }
			],
			{ yaw: -1.532813718098323, pitch: 0.3136986409540281, fov: 1.4774772949317303 }
		);

		// Scene: 0-foyer5bhk
		create360scene(
			'0-foyer5bhk',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 0.003235594505774486, pitch: -3.552713678800501e-15, fov: 1.4774772949317303 }
		);

		// Scene: 1-living-room5bhk
		create360scene(
			'1-living-room5bhk',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 0.0025908801659184633, pitch: 0.11281160709092575, fov: 1.4360600272292305 }
		);

		// Scene: 2-seatinglounge
		create360scene(
			'2-seatinglounge',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 3.1363941516091955, pitch: -0.008060354818416826, fov: 1.4774772949317303 }
		);

		// Scene: 3-deck5bhk
		create360scene(
			'3-deck5bhk',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 0.008416358277655078, pitch: 0.00258683043844421, fov: 1.4774772949317303 }
		);

		// Scene: 4-kitchen
		create360scene(
			'4-kitchen',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: -1.5899109572034824, pitch: 0.09895204456180551, fov: 1.4774772949317303 }
		);

		// Scene: 6-dinningarea
		create360scene(
			'6-dinningarea',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: -0.014667210791907692, pitch: 0.02313506782899566, fov: 1.4200995772540537 }
		);

		// Scene: 5-utility5bhk
		create360scene(
			'5-utility5bhk',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 0, pitch: 0, fov: 1.5707963267948966 }
		);

		// Scene: 7-poojaroom
		create360scene(
			'7-poojaroom',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 1.5711224094755671, pitch: 0, fov: 1.4774772949317303 }
		);

		// Scene: 8-family5bhk
		create360scene(
			'8-family5bhk',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 0.00022890590840596303, pitch: 0.001177521237726964, fov: 1.4774772949317303 }
		);

		// Scene: 9-flexroom
		create360scene(
			'9-flexroom',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 3.1393584975891553, pitch: 0.0024475747300769513, fov: 1.4195933434400527 }
		);

		// Scene: 10-masterbedroom
		create360scene(
			'10-masterbedroom',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: -3.1354888672329952, pitch: -0.0003804875602373414, fov: 1.4774772949317303 }
		);

		// Scene: 11-masterwardrobe
		create360scene(
			'11-masterwardrobe',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: -0.005440502704830763, pitch: 0.003068559558320416, fov: 1.4774772949317303 }
		);

		// Scene: 13-guestroom1
		create360scene(
			'13-guestroom1',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 0.38904060307542565, pitch: 0, fov: 1.4050701919942006 }
		);

		// Scene: 12-masterwashroom
		create360scene(
			'12-masterwashroom',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: -1.5690683611014542, pitch: 0.002547373046436263, fov: 1.4774772949317303 }
		);

		// Scene: 15-5bhkguestwashroom1
		create360scene(
			'15-5bhkguestwashroom1',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 1.5824861363492708, pitch: 0, fov: 1.4774772949317303 }
		);

		// Scene: 14-guestwardrobe1
		create360scene(
			'14-guestwardrobe1',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 1.5636810359731133, pitch: 0, fov: 1.4774772949317303 }
		);

		// Scene: 16-kidsroom
		create360scene(
			'16-kidsroom',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 0.0009520359650210253, pitch: 0.0022366170540415453, fov: 1.4774772949317303 }
		);

		// Scene: 17-kidswashroom
		create360scene(
			'17-kidswashroom',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 2.2026893116877915, pitch: 0.0008650400892449284, fov: 1.4774772949317303 }
		);

		// Scene: 19-guestwardrobe2
		create360scene(
			'19-guestwardrobe2',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: -2.3758563913578232, pitch: 0.010457206337230573, fov: 1.4774772949317303 }
		);

		// Scene: 18-guestroom2_5bhk
		create360scene(
			'18-guestroom2_5bhk',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: -0.004957468131904008, pitch: 0.01934127423617582, fov: 1.4774772949317303 }
		);

		// Scene: 21-5bhkcommon-washroom
		create360scene(
			'21-5bhkcommon-washroom',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: -1.5746849998114332, pitch: 0.7020357417996195, fov: 1.4774772949317303 }
		);

		// Scene: 20-guestwashroom2
		create360scene(
			'20-guestwashroom2',
			[
				{ tileSize: 256, size: 256, fallbackOnly: true },
				{ tileSize: 512, size: 512 },
				{ tileSize: 512, size: 1024 },
				{ tileSize: 512, size: 2048 }
			],
			{ yaw: 0.744159606452726, pitch: -0.005589792805574589, fov: 1.4774772949317303 }
		);

		createHotspot(
			allScenes['0-foyer'].scene,
			{ yaw: 2.1375300792170453, pitch: -0.11627514416643159 },
			allScenes['1-livingroom'].scene,
			'LivingRoom',
			'1-livingroom'
		);

		createHotspot(
			allScenes['1-livingroom'].scene,
			{ yaw: -2.746961645755448, pitch: -0.018517236996281383 },
			allScenes['6-family'].scene,
			'Family',
			'6-family'
		);

		createHotspot(
			allScenes['1-livingroom'].scene,
			{ yaw: -1.945892840244646, pitch: -0.01250443248304478 },
			allScenes['0-foyer'].scene,
			'Foyer',
			'0-foyer'
		);

		createHotspot(
			allScenes['1-livingroom'].scene,
			{ yaw: -2.3624215868829648, pitch: -0.050942103212761936 },
			allScenes['4-kitchen4bhk'].scene,
			'Kitchen4Bhk',
			'4-kitchen4bhk'
		);

		createHotspot(
			allScenes['1-livingroom'].scene,
			{ yaw: 2.938104446870339, pitch: -0.039868944542897466 },
			allScenes['3-dinning'].scene,
			'Dinning',
			'3-dinning'
		);

		createHotspot(
			allScenes['2-deck'].scene,
			{ yaw: -1.1864322303964023, pitch: -0.013532881103680339 },
			allScenes['0-foyer'].scene,
			'Foyer',
			'0-foyer'
		);

		createHotspot(
			allScenes['2-deck'].scene,
			{ yaw: -2.3993458125313367, pitch: 0.019691592088422638 },
			allScenes['3-dinning'].scene,
			'Dinning',
			'3-dinning'
		);

		createHotspot(
			allScenes['2-deck'].scene,
			{ yaw: -1.7865690684404534, pitch: -0.047796445631838935 },
			allScenes['4-kitchen4bhk'].scene,
			'Kitchen4Bhk',
			'4-kitchen4bhk'
		);

		createHotspot(
			allScenes['2-deck'].scene,
			{ yaw: -1.556455059924943, pitch: -0.0010259485921793043 },
			allScenes['1-livingroom'].scene,
			'LivingRoom',
			'1-livingroom'
		);

		createHotspot(
			allScenes['3-dinning'].scene,
			{ yaw: 0.615145085044146, pitch: 0.02616804361502645 },
			allScenes['1-livingroom'].scene,
			'LivingRoom',
			'1-livingroom'
		);

		createHotspot(
			allScenes['3-dinning'].scene,
			{ yaw: -1.2831376327106359, pitch: 0.04500022605877518 },
			allScenes['4-kitchen4bhk'].scene,
			'Kitchen4Bhk',
			'4-kitchen4bhk'
		);

		createHotspot(
			allScenes['3-dinning'].scene,
			{ yaw: -2.6834655517500714, pitch: 0.004577913871454342 },
			allScenes['6-family'].scene,
			'Family',
			'6-family'
		);

		createHotspot(
			allScenes['3-dinning'].scene,
			{ yaw: -0.4618739180452458, pitch: 0.021244435195123756 },
			allScenes['0-foyer'].scene,
			'Foyer',
			'0-foyer'
		);

		createHotspot(
			allScenes['4-kitchen4bhk'].scene,
			{ yaw: 1.9955654665720246, pitch: -0.06062059887901583 },
			allScenes['5-utility4bhk'].scene,
			'Utility4Bhk',
			'5-utility4bhk'
		);

		createHotspot(
			allScenes['4-kitchen4bhk'].scene,
			{ yaw: -1.2106899136359122, pitch: -0.06065251405108896 },
			allScenes['1-livingroom'].scene,
			'LivingRoom',
			'1-livingroom'
		);

		createHotspot(
			allScenes['4-kitchen4bhk'].scene,
			{ yaw: -1.921192465678505, pitch: -0.05306975179664519 },
			allScenes['0-foyer'].scene,
			'Foyer',
			'0-foyer'
		);

		createHotspot(
			allScenes['5-utility4bhk'].scene,
			{ yaw: -1.0653082663656797, pitch: -0.047436257206301846 },
			allScenes['4-kitchen4bhk'].scene,
			'Kitchen4Bhk',
			'4-kitchen4bhk'
		);

		createHotspot(
			allScenes['5-utility4bhk'].scene,
			{ yaw: -1.3016347141132982, pitch: -0.03780145993512818 },
			allScenes['1-livingroom'].scene,
			'LivingRoom',
			'1-livingroom'
		);

		createHotspot(
			allScenes['6-family'].scene,
			{ yaw: -1.9306613937229269, pitch: -0.08889997736826416 },
			allScenes['13-kidroom'].scene,
			'KidRoom',
			'13-kidroom'
		);

		createHotspot(
			allScenes['6-family'].scene,
			{ yaw: 1.979121143235596, pitch: -0.0760965888055054 },
			allScenes['7-masterbed4bhkph'].scene,
			'MasterBed4BhkPH',
			'7-masterbed4bhkph'
		);

		createHotspot(
			allScenes['6-family'].scene,
			{ yaw: 2.4309741205923743, pitch: -0.1465542783115268 },
			allScenes['10-guestroom014bhk'].scene,
			'GuestRoom014bhk',
			'10-guestroom014bhk'
		);

		createHotspot(
			allScenes['6-family'].scene,
			{ yaw: -2.4300585524334366, pitch: -0.10729125151671681 },
			allScenes['15-guestroom2'].scene,
			'GuestRoom2',
			'15-guestroom2'
		);

		createHotspot(
			allScenes['6-family'].scene,
			{ yaw: 0.9153693222131292, pitch: -0.1419522135945961 },
			allScenes['1-livingroom'].scene,
			'LivingRoom',
			'1-livingroom'
		);

		createHotspot(
			allScenes['7-masterbed4bhkph'].scene,
			{ yaw: -0.761104394755197, pitch: -0.10403522908446128 },
			allScenes['8-masterwardrobe'].scene,
			'MasterWardrobe',
			'8-masterwardrobe'
		);

		createHotspot(
			allScenes['7-masterbed4bhkph'].scene,
			{ yaw: -0.9625168351362223, pitch: -0.19367814414646745 },
			allScenes['6-family'].scene,
			'Family',
			'6-family'
		);

		createHotspot(
			allScenes['7-masterbed4bhkph'].scene,
			{ yaw: -1.6130881409634306, pitch: -0.13827071876436392 },
			allScenes['1-livingroom'].scene,
			'LivingRoom',
			'1-livingroom'
		);

		createHotspot(
			allScenes['8-masterwardrobe'].scene,
			{ yaw: 1.6535492321284728, pitch: -0.21006078708938425 },
			allScenes['1-livingroom'].scene,
			'LivingRoom',
			'1-livingroom'
		);

		createHotspot(
			allScenes['8-masterwardrobe'].scene,
			{ yaw: 1.3440521290900538, pitch: -0.16193809231187117 },
			allScenes['6-family'].scene,
			'Family',
			'6-family'
		);

		createHotspot(
			allScenes['8-masterwardrobe'].scene,
			{ yaw: -0.9172194990801685, pitch: -0.11508678047253085 },
			allScenes['7-masterbed4bhkph'].scene,
			'MasterBed4BhkPH',
			'7-masterbed4bhkph'
		);

		createHotspot(
			allScenes['9-masterwashroom'].scene,
			{ yaw: 0.6774363283650366, pitch: -0.10572474749241678 },
			allScenes['1-livingroom'].scene,
			'LivingRoom',
			'1-livingroom'
		);

		createHotspot(
			allScenes['9-masterwashroom'].scene,
			{ yaw: 2.3971239518872665, pitch: -0.24259588526967946 },
			allScenes['7-masterbed4bhkph'].scene,
			'MasterBed4BhkPH',
			'7-masterbed4bhkph'
		);

		createHotspot(
			allScenes['9-masterwashroom'].scene,
			{ yaw: 1.4564197354957908, pitch: -0.12093584936041069 },
			allScenes['8-masterwardrobe'].scene,
			'MasterWardrobe',
			'8-masterwardrobe'
		);

		createHotspot(
			allScenes['10-guestroom014bhk'].scene,
			{ yaw: -2.1566440218106813, pitch: -0.0885403900833488 },
			allScenes['11-guestwardrobe1'].scene,
			'GuestWardrobe1',
			'11-guestwardrobe1'
		);

		createHotspot(
			allScenes['10-guestroom014bhk'].scene,
			{ yaw: -2.7294211537554993, pitch: -0.1539895435517895 },
			allScenes['1-livingroom'].scene,
			'LivingRoom',
			'1-livingroom'
		);

		createHotspot(
			allScenes['11-guestwardrobe1'].scene,
			{ yaw: -2.131615846972444, pitch: -0.06450909324161458 },
			allScenes['10-guestroom014bhk'].scene,
			'GuestRoom014bhk',
			'10-guestroom014bhk'
		);

		createHotspot(
			allScenes['11-guestwardrobe1']?.scene,
			{ yaw: 2.8096997410310562, pitch: -0.08616276721563665 },
			allScenes['12-guestwashroom1'].scene,
			'GuestWashroom1',
			'12-guestwashroom1'
		);

		createHotspot(
			allScenes['12-guestwashroom1'].scene,
			{ yaw: -0.2009928003432364, pitch: -0.14399616037813523 },
			allScenes['11-guestwardrobe1'].scene,
			'GuestWardrobe1',
			'11-guestwardrobe1'
		);

		createHotspot(
			allScenes['12-guestwashroom1'].scene,
			{ yaw: -1.3714710813054527, pitch: -0.08968042205104254 },
			allScenes['10-guestroom014bhk'].scene,
			'GuestRoom014bhk',
			'10-guestroom014bhk'
		);

		createHotspot(
			allScenes['13-kidroom'].scene,
			{ yaw: 1.315352591805219, pitch: -0.04187266150332292 },
			allScenes['6-family'].scene,
			'Family',
			'6-family'
		);

		createHotspot(
			allScenes['13-kidroom'].scene,
			{ yaw: 0.858483369138062, pitch: -0.11502164303173856 },
			allScenes['1-livingroom'].scene,
			'LivingRoom',
			'1-livingroom'
		);

		createHotspot(
			allScenes['13-kidroom'].scene,
			{ yaw: 0.31512698528533, pitch: -0.22408781799118493 },
			allScenes['14-kidswashroom'].scene,
			'KidsWashRoom',
			'14-kidswashroom'
		);

		createHotspot(
			allScenes['14-kidswashroom'].scene,
			{ yaw: -2.8070903607557227, pitch: -0.10417329006397935 },
			allScenes['13-kidroom'].scene,
			'KidRoom',
			'13-kidroom'
		);

		createHotspot(
			allScenes['14-kidswashroom'].scene,
			{ yaw: 1.9751464380477302, pitch: 0.03493013337116402 },
			allScenes['6-family'].scene,
			'Family',
			'6-family'
		);

		createHotspot(
			allScenes['14-kidswashroom'].scene,
			{ yaw: 2.4666207851772377, pitch: -0.452094016103711 },
			allScenes['1-livingroom'].scene,
			'LivingRoom',
			'1-livingroom'
		);

		createHotspot(
			allScenes['15-guestroom2'].scene,
			{ yaw: -0.6225644122506289, pitch: -0.044637272363882374 },
			allScenes['16-guestwardrobe2'].scene,
			'GuestWardrobe2',
			'16-guestwardrobe2'
		);

		createHotspot(
			allScenes['15-guestroom2'].scene,
			{ yaw: -1.3809832370058395, pitch: -0.12973779825676068 },
			allScenes['15-guestroom2'].scene,
			'GuestRoom2',
			'15-guestroom2'
		);

		createHotspot(
			allScenes['15-guestroom2'].scene,
			{ yaw: -1.055155696448118, pitch: -0.5006389472718951 },
			allScenes['1-livingroom'].scene,
			'LivingRoom',
			'1-livingroom'
		);

		createHotspot(
			allScenes['16-guestwardrobe2'].scene,
			{ yaw: 2.475115523833386, pitch: -0.05216367325919613 },
			allScenes['15-guestroom2'].scene,
			'GuestRoom2',
			'15-guestroom2'
		);

		createHotspot(
			allScenes['16-guestwardrobe2'].scene,
			{ yaw: 1.5261714005013163, pitch: -0.08240379909032036 },
			allScenes['17-guestwashroom2'].scene,
			'GuestWashroom2',
			'17-guestwashroom2'
		);

		createHotspot(
			allScenes['16-guestwardrobe2'].scene,
			{ yaw: -1.6367586411706014, pitch: -0.16748601612728997 },
			allScenes['1-livingroom'].scene,
			'LivingRoom',
			'1-livingroom'
		);

		createHotspot(
			allScenes['17-guestwashroom2'].scene,
			{ yaw: -2.58773731601951, pitch: -0.09841268462507458 },
			allScenes['15-guestroom2'].scene,
			'GuestRoom2',
			'15-guestroom2'
		);

		createHotspot(
			allScenes['17-guestwashroom2'].scene,
			{ yaw: -0.9645352120665045, pitch: -0.3553582100324846 },
			allScenes['6-family'].scene,
			'Family',
			'6-family'
		);

		createHotspot(
			allScenes['17-guestwashroom2'].scene,
			{ yaw: -0.3006322364294096, pitch: -0.6729545972909712 },
			allScenes['1-livingroom'].scene,
			'LivingRoom',
			'1-livingroom'
		);

		createHotspot(
			allScenes['18-commonwashroom'].scene,
			{ yaw: 0.677669675605161, pitch: -0.28451618052049277 },
			allScenes['6-family'].scene,
			'Family',
			'6-family'
		);

		createHotspot(
			allScenes['18-commonwashroom'].scene,
			{ yaw: -3.0719586397389875, pitch: -0.19340632833516835 },
			allScenes['1-livingroom'].scene,
			'LivingRoom',
			'1-livingroom'
		);

		// ========================================
		// 5BHK SCENES AND HOTSPOTS (data1.js)
		// ========================================

		createHotspot(
			allScenes['0-foyer5bhk'].scene,
			{ yaw: 0.8950288279090355, pitch: -0.07844602478011353 },
			allScenes['1-living-room5bhk'].scene,
			'Living Room5BHK',
			'1-living-room5bhk'
		);

		createHotspot(
			allScenes['0-foyer5bhk'].scene,
			{ yaw: 2.2283189195572595, pitch: 0.020675531969853722 },
			allScenes['2-seatinglounge'].scene,
			'SeatingLounge',
			'2-seatinglounge'
		);

		createHotspot(
			allScenes['0-foyer5bhk'].scene,
			{ yaw: 1.5903034075965632, pitch: -0.035459437742876077 },
			allScenes['3-deck5bhk'].scene,
			'Deck5Bhk',
			'3-deck5bhk'
		);

		createHotspot(
			allScenes['0-foyer5bhk'].scene,
			{ yaw: 0.37955003161940937, pitch: -0.1204586658013973 },
			allScenes['4-kitchen'].scene,
			'Kitchen',
			'4-kitchen'
		);

		createHotspot(
			allScenes['1-living-room5bhk'].scene,
			{ yaw: 0.17850797354463133, pitch: -0.032291020550271554 },
			allScenes['2-seatinglounge'].scene,
			'SeatingLounge',
			'2-seatinglounge'
		);

		createHotspot(
			allScenes['1-living-room5bhk'].scene,
			{ yaw: -1.0926822085903947, pitch: -0.10042065613165363 },
			allScenes['0-foyer5bhk'].scene,
			'Foyer5Bhk',
			'0-foyer5bhk'
		);

		createHotspot(
			allScenes['1-living-room5bhk'].scene,
			{ yaw: -0.6750053430356147, pitch: 0.027594188205769754 },
			allScenes['4-kitchen'].scene,
			'Kitchen',
			'4-kitchen'
		);

		createHotspot(
			allScenes['1-living-room5bhk'].scene,
			{ yaw: -0.34793368960645665, pitch: -0.10153069146839755 },
			allScenes['10-masterbedroom'].scene,
			'MasterBedRoom',
			'10-masterbedroom'
		);

		createHotspot(
			allScenes['2-seatinglounge'].scene,
			{ yaw: -1.8313199664953341, pitch: -0.035089335608423866 },
			allScenes['0-foyer5bhk'].scene,
			'Foyer5Bhk',
			'0-foyer5bhk'
		);

		createHotspot(
			allScenes['2-seatinglounge'].scene,
			{ yaw: -1.4168427207881962, pitch: -0.03508932814788501 },
			allScenes['4-kitchen'].scene,
			'Kitchen',
			'4-kitchen'
		);

		createHotspot(
			allScenes['2-seatinglounge'].scene,
			{ yaw: -0.6266276677510998, pitch: -0.11225489979858949 },
			allScenes['10-masterbedroom'].scene,
			'MasterBedRoom',
			'10-masterbedroom'
		);

		createHotspot(
			allScenes['3-deck5bhk'].scene,
			{ yaw: -1.596304027572554, pitch: -0.024922789577800586 },
			allScenes['0-foyer5bhk'].scene,
			'Foyer5Bhk',
			'0-foyer5bhk'
		);

		createHotspot(
			allScenes['3-deck5bhk'].scene,
			{ yaw: -2.16900812385531, pitch: 0.08809467019526807 },
			allScenes['1-living-room5bhk'].scene,
			'Living Room5BHK',
			'1-living-room5bhk'
		);

		createHotspot(
			allScenes['3-deck5bhk'].scene,
			{ yaw: -0.7754998795708108, pitch: 0.004532104023404315 },
			allScenes['2-seatinglounge'].scene,
			'SeatingLounge',
			'2-seatinglounge'
		);

		createHotspot(
			allScenes['3-deck5bhk'].scene,
			{ yaw: -1.357745108192546, pitch: -0.027450349049374267 },
			allScenes['4-kitchen'].scene,
			'Kitchen',
			'4-kitchen'
		);

		createHotspot(
			allScenes['3-deck5bhk'].scene,
			{ yaw: -0.9661459218523554, pitch: -0.07468928114955986 },
			allScenes['10-masterbedroom'].scene,
			'MasterBedRoom',
			'10-masterbedroom'
		);

		createHotspot(
			allScenes['4-kitchen'].scene,
			{ yaw: -0.5950326760416491, pitch: -0.03980199175734711 },
			allScenes['6-dinningarea'].scene,
			'DinningArea',
			'6-dinningarea'
		);

		createHotspot(
			allScenes['4-kitchen'].scene,
			{ yaw: 1.535895304387557, pitch: -0.03387534632737399 },
			allScenes['2-seatinglounge'].scene,
			'SeatingLounge',
			'2-seatinglounge'
		);

		createHotspot(
			allScenes['5-utility5bhk'].scene,
			{ yaw: -1.1910879670808896, pitch: -0.021848023330068855 },
			allScenes['1-living-room5bhk'].scene,
			'Living Room5BHK',
			'1-living-room5bhk'
		);

		createHotspot(
			allScenes['5-utility5bhk'].scene,
			{ yaw: -0.7183456734089635, pitch: -0.06577134937112206 },
			allScenes['4-kitchen'].scene,
			'Kitchen',
			'4-kitchen'
		);

		createHotspot(
			allScenes['5-utility5bhk'].scene,
			{ yaw: -1.9951792315065635, pitch: -0.036903837279766094 },
			allScenes['10-masterbedroom'].scene,
			'MasterBedRoom',
			'10-masterbedroom'
		);

		createHotspot(
			allScenes['6-dinningarea'].scene,
			{ yaw: -1.4180078765605657, pitch: -0.12112448626209726 },
			allScenes['4-kitchen'].scene,
			'Kitchen',
			'4-kitchen'
		);

		createHotspot(
			allScenes['6-dinningarea'].scene,
			{ yaw: 3.0012705088798945, pitch: -0.1323999802654079 },
			allScenes['7-poojaroom'].scene,
			'PoojaRoom',
			'7-poojaroom'
		);

		createHotspot(
			allScenes['7-poojaroom'].scene,
			{ yaw: -1.44032862581615, pitch: -0.08868807714291727 },
			allScenes['6-dinningarea'].scene,
			'DinningArea',
			'6-dinningarea'
		);

		createHotspot(
			allScenes['7-poojaroom'].scene,
			{ yaw: -2.723549221052462, pitch: -0.024772577080332425 },
			allScenes['4-kitchen'].scene,
			'Kitchen',
			'4-kitchen'
		);

		createHotspot(
			allScenes['7-poojaroom'].scene,
			{ yaw: -0.04811151904334032, pitch: -0.07043748447571296 },
			allScenes['8-family5bhk'].scene,
			'Family5BHK',
			'8-family5bhk'
		);

		createHotspot(
			allScenes['8-family5bhk'].scene,
			{ yaw: -1.218754241417173, pitch: -0.009381592342776912 },
			allScenes['16-kidsroom'].scene,
			'KidsRoom',
			'16-kidsroom'
		);

		createHotspot(
			allScenes['8-family5bhk'].scene,
			{ yaw: -1.5513653296608005, pitch: -0.11850778732983969 },
			allScenes['21-5bhkcommon-washroom'].scene,
			'5bhkCommon washroom',
			'21-5bhkcommon-washroom'
		);

		createHotspot(
			allScenes['8-family5bhk'].scene,
			{ yaw: -2.4339844537873, pitch: -0.05804643007386545 },
			allScenes['1-living-room5bhk'].scene,
			'Living Room5BHK',
			'1-living-room5bhk'
		);

		createHotspot(
			allScenes['8-family5bhk'].scene,
			{ yaw: 1.2175251415944324, pitch: -0.09157035854783757 },
			allScenes['18-guestroom2_5bhk'].scene,
			'GuestRoom2_5BHK',
			'18-guestroom2_5bhk'
		);

		createHotspot(
			allScenes['9-flexroom'].scene,
			{ yaw: -0.2527567196701419, pitch: -0.2635631375014498 },
			allScenes['1-living-room5bhk'].scene,
			'Living Room5BHK',
			'1-living-room5bhk'
		);

		createHotspot(
			allScenes['9-flexroom'].scene,
			{ yaw: -0.9472423311666418, pitch: -0.09813264120863963 },
			allScenes['10-masterbedroom'].scene,
			'MasterBedRoom',
			'10-masterbedroom'
		);

		createHotspot(
			allScenes['10-masterbedroom'].scene,
			{ yaw: -2.4712653239474314, pitch: 0.0050272185803983405 },
			allScenes['12-masterwashroom'].scene,
			'MasterWashRoom',
			'12-masterwashroom'
		);

		createHotspot(
			allScenes['10-masterbedroom'].scene,
			{ yaw: -1.40488905078106, pitch: -0.13752400215304306 },
			allScenes['1-living-room5bhk'].scene,
			'Living Room5BHK',
			'1-living-room5bhk'
		);

		createHotspot(
			allScenes['11-masterwardrobe'].scene,
			{ yaw: 3.138231569553078, pitch: -0.0010259855879866109 },
			allScenes['10-masterbedroom'].scene,
			'MasterBedRoom',
			'10-masterbedroom'
		);

		createHotspot(
			allScenes['11-masterwardrobe'].scene,
			{ yaw: 2.463683147464196, pitch: 0.00827498557936579 },
			allScenes['1-living-room5bhk'].scene,
			'Living Room5BHK',
			'1-living-room5bhk'
		);

		createHotspot(
			allScenes['12-masterwashroom'].scene,
			{ yaw: 1.4168632191635666, pitch: -0.35412006025275744 },
			allScenes['10-masterbedroom'].scene,
			'MasterBedRoom',
			'10-masterbedroom'
		);

		createHotspot(
			allScenes['12-masterwashroom'].scene,
			{ yaw: 2.6179096520448155, pitch: -0.0653652168579697 },
			allScenes['1-living-room5bhk'].scene,
			'Living Room5BHK',
			'1-living-room5bhk'
		);

		createHotspot(
			allScenes['13-guestroom1'].scene,
			{ yaw: -0.39781599444700433, pitch: -0.07772555734579356 },
			allScenes['14-guestwardrobe1'].scene,
			'GuestWardrobe1',
			'14-guestwardrobe1'
		);

		createHotspot(
			allScenes['13-guestroom1'].scene,
			{ yaw: -1.0754750656653869, pitch: -0.06650881089733574 },
			allScenes['1-living-room5bhk'].scene,
			'Living Room5BHK',
			'1-living-room5bhk'
		);

		createHotspot(
			allScenes['14-guestwardrobe1'].scene,
			{ yaw: 3.0975542266913294, pitch: 0.15970723959839717 },
			allScenes['13-guestroom1'].scene,
			'GuestRoom1',
			'13-guestroom1'
		);

		createHotspot(
			allScenes['14-guestwardrobe1'].scene,
			{ yaw: -2.2678452098122364, pitch: -0.012440157977303912 },
			allScenes['1-living-room5bhk'].scene,
			'Living Room5BHK',
			'1-living-room5bhk'
		);

		createHotspot(
			allScenes['15-5bhkguestwashroom1'].scene,
			{ yaw: 2.836800690047051, pitch: -0.13144704116685801 },
			allScenes['13-guestroom1'].scene,
			'GuestRoom1',
			'13-guestroom1'
		);

		createHotspot(
			allScenes['15-5bhkguestwashroom1'].scene,
			{ yaw: -2.308998598722109, pitch: -0.2946231716716383 },
			allScenes['1-living-room5bhk'].scene,
			'Living Room5BHK',
			'1-living-room5bhk'
		);

		createHotspot(
			allScenes['16-kidsroom'].scene,
			{ yaw: 1.730240955858375, pitch: -0.017372984704017824 },
			allScenes['8-family5bhk'].scene,
			'Family5BHK',
			'8-family5bhk'
		);

		createHotspot(
			allScenes['16-kidsroom'].scene,
			{ yaw: 2.945194566697398, pitch: -0.2283636131840261 },
			allScenes['1-living-room5bhk'].scene,
			'Living Room5BHK',
			'1-living-room5bhk'
		);

		createHotspot(
			allScenes['16-kidsroom'].scene,
			{ yaw: 2.5455739908571555, pitch: 0.02895297234681493 },
			allScenes['17-kidswashroom'].scene,
			'KidsWashRoom',
			'17-kidswashroom'
		);

		createHotspot(
			allScenes['17-kidswashroom'].scene,
			{ yaw: -1.7101788106406595, pitch: -0.22349881433484242 },
			allScenes['16-kidsroom'].scene,
			'KidsRoom',
			'16-kidsroom'
		);

		createHotspot(
			allScenes['17-kidswashroom'].scene,
			{ yaw: 0.20714938756380263, pitch: -0.1884204550910411 },
			allScenes['1-living-room5bhk'].scene,
			'Living Room5BHK',
			'1-living-room5bhk'
		);

		createHotspot(
			allScenes['18-guestroom2_5bhk'].scene,
			{ yaw: -2.3125422245193477, pitch: 0.05415313236590258 },
			allScenes['19-guestwardrobe2'].scene,
			'GuestWardrobe2',
			'19-guestwardrobe2'
		);

		createHotspot(
			allScenes['18-guestroom2_5bhk'].scene,
			{ yaw: -2.6875858090730986, pitch: -0.01816288776343633 },
			allScenes['1-living-room5bhk'].scene,
			'Living Room5BHK',
			'1-living-room5bhk'
		);

		createHotspot(
			allScenes['18-guestroom2_5bhk'].scene,
			{ yaw: -1.8316735177878378, pitch: -0.04187900457774241 },
			allScenes['8-family5bhk'].scene,
			'Family5BHK',
			'8-family5bhk'
		);

		createHotspot(
			allScenes['19-guestwardrobe2'].scene,
			{ yaw: 0.41903142467987564, pitch: -0.08182536131322848 },
			allScenes['18-guestroom2_5bhk'].scene,
			'GuestRoom2_5BHK',
			'18-guestroom2_5bhk'
		);

		createHotspot(
			allScenes['20-guestwashroom2'].scene,
			{ yaw: 1.9564020863192377, pitch: -0.10911665721639707 },
			allScenes['18-guestroom2_5bhk'].scene,
			'GuestRoom2_5BHK',
			'18-guestroom2_5bhk'
		);

		createHotspot(
			allScenes['20-guestwashroom2'].scene,
			{ yaw: -3.099347314497045, pitch: -0.31595349563642117 },
			allScenes['1-living-room5bhk'].scene,
			'Living Room5BHK',
			'1-living-room5bhk'
		);

		createHotspot(
			allScenes['21-5bhkcommon-washroom'].scene,
			{ yaw: 1.2526531457288979, pitch: -0.0036298478973897375 },
			allScenes['8-family5bhk'].scene,
			'Family5BHK',
			'8-family5bhk'
		);

		createHotspot(
			allScenes['21-5bhkcommon-washroom'].scene,
			{ yaw: 2.576535503721826, pitch: -0.07847406562927972 },
			allScenes['1-living-room5bhk'].scene,
			'Living Room5BHK',
			'1-living-room5bhk'
		);

		var pano = document.getElementById('pano');
		// hotspots
		//

		// Hotspots generated from APP_DATA linkHotspots

		// -- generated hotspots
		//
		//

		pano.addEventListener('click', function (e) {
			// var imgHotspot = document.createElement('img');
			// imgHotspot.src = '';
			// imgHotspot.classList.add('hotspot');

			// // http://www.marzipano.net/reference/Viewer.html#view dapat dari view di index.js
			var view = viewer.view();
			var loc = view.screenToCoordinates({ x: e.clientX, y: e.clientY });
			var position = { yaw: loc._yaw, pitch: loc._pitch };
			// http://www.marzipano.net/reference/Viewer.html#scene dapat dari scene
			console.log(view.screenToCoordinates({ x: e.clientX, y: e.clientY }));
			console.log(view._yaw + ' - ' + view._pitch);
			window.hotspotData = JSON.stringify(view.screenToCoordinates({ x: e.clientX, y: e.clientY }));
			var sceness = viewer.scene();
			// sceness.hotspotContainer().createHotspot(imgHotspot, position);
		});
		hotspotName.set('0-foyer');
		unsubscribeHotSpot = hotspotName.subscribe((changedHotspot) => {
			console.log('$currentUI.interiors', $currentUI.interiors, changedHotspot);
			if ($currentUI.interiors) {
				allScenes[changedHotspot].scene.switchTo();
				window.view = allScenes[changedHotspot].view;
				// window.dragControlMethod.addEventListener('parameterDynamics', resetArrowValues);
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
						alert(`${finalTextToCopy}`);
					},
					() => {
						alert('Failed to copy hotspot info');
					}
				);
			}
		});
	});

	function resetArrowValues() {
		document.querySelectorAll('.info-hotspot').forEach((e) => {
			console.log('LISTENER');
			setTimeout(() => {
				e.style.transform = resetNegativeTranslations(
					e.style.transform,
					e.querySelector('.hotspot')
				);
			}, 500);
		});
	}
	onDestroy(() => {
		if (unsubscribeHotSpot) unsubscribeHotSpot();
	});
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
</script>

{#if !$startTour}
	<div class="dollhouse-wrapper z-[9]" transition:fade>
		{#if $currentbhk === '4bhk'}
			<img src={dollhouse} id="dollhouse" class="" alt="dollhouse 3bhksmall" />
		{:else if $currentbhk === '5bhk'}
			<img src={dollhouse1} id="dollhouse" class="" alt="dollhouse 3bhkmedium" />
		{/if}
		<button
			id="start-tour"
			class="primary-btn safari-btn mx-auto mt-[-1rem]"
			on:click={() => ($startTour = true)}
		>
			<img id="startTourImg" src={startTourImg} alt="start tour" />
			Start Tour
		</button>
	</div>
{/if}

<div class="bhk-selector fixed right-5 top-8 z-[999]">
	<div class="relative w-fit">
		<div class=" ">
			<div class="rounded-xl bg-white p-2">
				<div class="    ">
					<div class="left-panel--header flex justify-between gap-[2rem] lg:gap-[5rem]">
						<div class="left-title flex items-center font-bold">
							<svg
								class="mr-2"
								width="17"
								height="17"
								viewBox="0 0 17 17"
								fill="none"
								xmlns="http://www.w3.org/2000/svg"
							>
								<path
									d="M8.68982 16.9237C6.64273 16.9237 4.59877 16.9144 2.55168 16.9299C2.14038 16.9331 1.87036 16.6185 1.87036 16.2479C1.8735 14.426 1.8735 12.6042 1.8735 10.7823C1.8735 9.80127 1.8735 8.82026 1.8735 7.83925C1.8735 7.78319 1.90176 7.70845 1.85152 7.67731C1.80129 7.64616 1.75419 7.71468 1.7071 7.74271C1.40255 7.93891 1.09799 8.13823 0.790301 8.33443C0.755764 8.35623 0.708668 8.38737 0.680411 8.37803C0.627036 8.35934 0.649014 8.29706 0.649014 8.25346C0.649014 7.61502 0.649014 6.97659 0.645874 6.33815C0.645874 6.23538 0.683551 6.17621 0.765183 6.12326C1.87664 5.3914 2.9881 4.65953 4.1027 3.93078C4.74634 3.50723 5.38998 3.0868 6.03363 2.66637C6.86879 2.11825 7.70709 1.57636 8.53912 1.02512C8.65529 0.947263 8.74634 0.950378 8.86251 1.02824C9.60348 1.5203 10.3476 2.00613 11.0917 2.49197C12.4512 3.38578 13.8138 4.27958 15.1733 5.17028C15.6569 5.48794 16.1372 5.8056 16.6239 6.11703C16.7181 6.17621 16.7558 6.24472 16.7526 6.35684C16.7463 6.98593 16.7495 7.61191 16.7495 8.241C16.7495 8.28771 16.7683 8.35 16.7275 8.3718C16.6773 8.40294 16.6333 8.34689 16.5925 8.31886C16.2817 8.11954 15.974 7.91711 15.6631 7.71779C15.6286 7.69599 15.5878 7.63994 15.5438 7.67108C15.5218 7.68665 15.525 7.74894 15.525 7.78942C15.525 9.21578 15.525 10.6421 15.525 12.0716C15.525 13.4668 15.5187 14.862 15.5281 16.2604C15.5313 16.6247 15.211 16.9237 14.8562 16.9237C12.7997 16.9175 10.7432 16.9237 8.68982 16.9237ZM8.70238 15.6562C10.4795 15.6562 12.2597 15.6562 14.0368 15.6562C14.244 15.6562 14.244 15.6562 14.244 15.4506C14.244 12.6135 14.244 9.77324 14.2471 6.9361C14.2471 6.82087 14.2032 6.7617 14.1121 6.70253C13.3649 6.21981 12.6207 5.73397 11.8798 5.24502C10.8594 4.57545 9.83582 3.90275 8.81855 3.23006C8.73064 3.17089 8.66785 3.174 8.58307 3.23317C8.18119 3.50412 7.77617 3.76572 7.37114 4.03044C6.00537 4.92425 4.63959 5.82117 3.27382 6.71187C3.18276 6.77104 3.15451 6.83644 3.15765 6.94233C3.16079 9.77013 3.16079 12.5979 3.16079 15.4257C3.16079 15.6593 3.16079 15.6593 3.40254 15.6593C5.16392 15.6562 6.93158 15.6562 8.70238 15.6562Z"
									fill="#5A4DE3"
								/>
								<path
									d="M11.1368 15.0858C11.0897 15.0858 11.0426 15.0858 10.9955 15.0858C10.6125 15.0858 10.4994 14.9861 10.4838 14.6155C10.4775 14.4847 10.399 14.494 10.3142 14.494C9.61719 14.494 8.92331 14.494 8.2263 14.494C7.84325 14.494 7.46334 14.4972 7.0803 14.4909C6.95471 14.4878 6.91703 14.5314 6.92017 14.6529C6.92331 14.9425 6.77261 15.0858 6.47747 15.0858C6.31421 15.0858 6.15408 15.0889 5.99082 15.0858C5.7679 15.0826 5.59835 14.9207 5.58266 14.6996C5.58266 14.6778 5.57952 14.6591 5.58266 14.6373C5.62661 14.3041 5.5544 14.0269 5.30008 13.7715C5.08658 13.5566 5.00809 13.2545 4.98611 12.9462C4.97983 12.8434 4.96413 12.7407 4.95785 12.6379C4.95471 12.5818 4.91704 12.56 4.87308 12.5351C4.4712 12.3265 4.30479 12.0088 4.36131 11.5479C4.40212 11.2022 4.73179 10.8565 5.10228 10.7693C5.60463 10.6541 6.13211 10.9998 6.23572 11.5012C6.27339 11.6787 6.24827 11.8624 6.29537 12.04C6.40526 12.4542 6.58422 12.5974 7.02065 12.5974C8.13838 12.5974 9.25926 12.5974 10.377 12.5974C10.8762 12.5974 11.0803 12.4137 11.1368 11.8375C11.1619 11.5884 11.1776 11.3455 11.3409 11.1368C11.5732 10.8378 11.8746 10.7133 12.2451 10.7631C12.5874 10.8098 12.8323 11.0122 12.9704 11.3143C13.1588 11.7285 13.0552 12.3078 12.5277 12.532C12.4461 12.5663 12.4524 12.6317 12.4461 12.7002C12.4241 13.0801 12.355 13.4507 12.107 13.7591C12.0473 13.8307 12.0002 13.9241 11.9249 13.9708C11.7648 14.0705 11.7459 14.2106 11.7553 14.3726C11.7616 14.5034 11.7585 14.6311 11.7585 14.7619C11.7553 14.9612 11.6392 15.0764 11.4351 15.0826C11.3346 15.0889 11.2341 15.0889 11.1368 15.0858Z"
									fill="#5A4DE3"
								/>
								<path
									d="M8.7684 8.36811C9.39634 8.36811 10.0243 8.37434 10.6522 8.36811C11.5784 8.35565 12.445 9.14357 12.4795 10.0623C12.4795 10.0872 12.4827 10.1152 12.4921 10.1402C12.5204 10.2305 12.4984 10.2647 12.3948 10.2492C12.0149 10.1869 11.6538 10.2585 11.3304 10.4609C10.9191 10.7163 10.7056 11.0963 10.6585 11.5727C10.6522 11.6288 10.6459 11.6849 10.6365 11.7409C10.5832 12.0368 10.5832 12.0368 10.288 12.0368C9.21424 12.0368 8.14359 12.0368 7.06981 12.0368C6.85631 12.0368 6.85945 12.0368 6.83747 11.8281C6.80294 11.498 6.77154 11.1679 6.56118 10.8876C6.20325 10.4142 5.74485 10.162 5.13261 10.246C5.04469 10.2585 5.01958 10.2305 5.02272 10.1495C5.05097 9.6699 5.23308 9.25258 5.56903 8.91312C5.8987 8.57988 6.30372 8.38057 6.77782 8.37122C7.4403 8.35877 8.10592 8.36811 8.7684 8.36811Z"
									fill="#5A4DE3"
								/>
							</svg>
							<span class="font-semibold">
								{#if $currentbhk === '4bhk'}4 BHK
								{:else if $currentbhk === '5bhk'}5 BHK
								{/if}
							</span>
						</div>
						<button
							on:click={() => {
								$isbhklist = !$isbhklist;
							}}
							class="ghost-btn close-btn !px-0 !py-0"
							id="minimize-toggle-vicinity"
						>
							{#if !$isbhklist}
								<img src={minimizeBtn} alt="minimize" />
							{/if}
							{#if $isbhklist}
								<img src={maximizeBtn} alt="maximize" />
							{/if}
						</button>
					</div>

					<div class={!$isbhklist ? 'block' : 'hidden'}>
						<div class="no-hovers pt-3">
							<div class=" ">
								<Accordion.Root class="w-full sm:max-w-full">
									<Accordion.Item value="item-12" class="none ">
										<Accordion.Trigger id="grolevel-fx">Select BHK Type</Accordion.Trigger>
									</Accordion.Item>
									<div class="flex flex-col gap-2 bg-white py-2">
										<button
											class={$currentbhk === '4bhk' ? 'active inner-modal-btn' : 'inner-modal-btn'}
											on:click={() => {
												currentbhk.set('4bhk');
												$hotspotName = '0-foyer';
												$startTour = false;
												$isbhklist = !$isbhklist;
											}}
										>
											4 BHK
										</button>

										<button
											class={$currentbhk === '5bhk' ? 'active inner-modal-btn' : 'inner-modal-btn'}
											on:click={() => {
												currentbhk.set('5bhk');
												$hotspotName = '0-foyer5bhk';
												$startTour = false;
												$isbhklist = !$isbhklist;
											}}
										>
											5 BHK
										</button>
									</div>
								</Accordion.Root>
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</div>

{#if $startTour}
	<div class="left-panel-wrapper">
		<div class="left-panel p-2">
			<div class="left-panel--header flex justify-between gap-[1rem] lg:gap-[3rem]">
				<div class="left-title flex items-center font-bold">
					<svg
						width="25"
						height="25"
						viewBox="0 0 32 31"
						class="mr-2"
						fill="none"
						xmlns="http://www.w3.org/2000/svg"
					>
						<path
							d="M15.7834 30.1853C11.9139 30.1853 8.05047 30.1677 4.18106 30.1971C3.40362 30.203 2.89323 29.6084 2.89323 28.9079C2.89917 25.4642 2.89917 22.0205 2.89917 18.5768C2.89917 16.7225 2.89917 14.8682 2.89917 13.0139C2.89917 12.9079 2.95258 12.7666 2.85763 12.7078C2.76267 12.6489 2.67365 12.7784 2.58463 12.8314C2.00897 13.2023 1.4333 13.579 0.851705 13.9499C0.786424 13.9911 0.697404 14.0499 0.643991 14.0323C0.543102 13.997 0.584645 13.8792 0.584645 13.7968C0.584645 12.59 0.584645 11.3833 0.57871 10.1765C0.57871 9.98225 0.649926 9.8704 0.804228 9.77033C2.9051 8.38696 5.00598 7.00359 7.11279 5.62611C8.3294 4.82552 9.54601 4.03082 10.7626 3.23612C12.3412 2.20007 13.9258 1.17578 15.4985 0.133843C15.7181 -0.0133237 15.8902 -0.00743706 16.1098 0.13973C17.5103 1.06982 18.9169 1.98815 20.3234 2.90647C22.8931 4.59594 25.4687 6.28542 28.0385 7.96901C28.9524 8.56945 29.8604 9.16989 30.7803 9.75856C30.9583 9.8704 31.0295 9.99991 31.0236 10.2118C31.0117 11.4009 31.0177 12.5842 31.0177 13.7733C31.0177 13.8616 31.0533 13.9793 30.9761 14.0205C30.8812 14.0794 30.7981 13.9734 30.7209 13.9204C30.1334 13.5437 29.5518 13.1611 28.9643 12.7843C28.899 12.7431 28.8218 12.6371 28.7388 12.696C28.6972 12.7254 28.7031 12.8432 28.7031 12.9197C28.7031 15.6158 28.7031 18.3119 28.7031 21.0139C28.7031 23.6511 28.6913 26.2883 28.7091 28.9315C28.715 29.6202 28.1097 30.1853 27.4391 30.1853C23.5518 30.1735 19.6646 30.1853 15.7834 30.1853ZM15.8071 27.7894C19.1661 27.7894 22.5311 27.7894 25.8901 27.7894C26.2818 27.7894 26.2818 27.7894 26.2818 27.4009C26.2818 22.0382 26.2818 16.6695 26.2877 11.3068C26.2877 11.0889 26.2046 10.9771 26.0325 10.8653C24.6201 9.95282 23.2136 9.0345 21.813 8.11029C19.8842 6.84465 17.9495 5.57313 16.0267 4.30161C15.8605 4.18976 15.7418 4.19565 15.5816 4.3075C14.8219 4.81964 14.0564 5.31412 13.2908 5.81448C10.7092 7.50396 8.12762 9.19932 5.54604 10.8829C5.37393 10.9948 5.32052 11.1184 5.32645 11.3185C5.33239 16.6636 5.33239 22.0087 5.33239 27.3538C5.33239 27.7953 5.33239 27.7953 5.78936 27.7953C9.11871 27.7894 12.4599 27.7894 15.8071 27.7894Z"
							fill="#5A4DE3"
						/>
						<path
							d="M20.4084 26.7112C20.3193 26.7112 20.2303 26.7112 20.1413 26.7112C19.4173 26.7112 19.2036 26.5229 19.1739 25.8223C19.1621 25.5751 19.0137 25.5928 18.8535 25.5928C17.536 25.5928 16.2244 25.5928 14.9069 25.5928C14.1829 25.5928 13.4648 25.5986 12.7407 25.5869C12.5034 25.581 12.4321 25.6634 12.4381 25.893C12.444 26.4404 12.1592 26.7112 11.6013 26.7112C11.2927 26.7112 10.99 26.7171 10.6814 26.7112C10.2601 26.7053 9.93958 26.3992 9.90991 25.9813C9.90991 25.9401 9.90397 25.9048 9.90991 25.8635C9.99299 25.2337 9.8565 24.7098 9.37579 24.2271C8.97223 23.8209 8.82386 23.2499 8.78232 22.6671C8.77045 22.4728 8.74078 22.2786 8.72891 22.0843C8.72297 21.9783 8.65176 21.9371 8.56867 21.89C7.80903 21.4956 7.49449 20.8952 7.60132 20.024C7.67847 19.3705 8.30161 18.7171 9.0019 18.5523C9.95145 18.3345 10.9485 18.9879 11.1443 19.9357C11.2155 20.2712 11.1681 20.6185 11.2571 20.9541C11.4648 21.737 11.8031 22.0078 12.628 22.0078C14.7407 22.0078 16.8594 22.0078 18.9722 22.0078C19.9158 22.0078 20.3015 21.6605 20.4084 20.5714C20.4558 20.1005 20.4855 19.6413 20.7941 19.2469C21.2333 18.6818 21.803 18.4463 22.5033 18.5405C23.1502 18.6288 23.6131 19.0115 23.8742 19.5825C24.2303 20.3654 24.0344 21.4603 23.0374 21.8842C22.8831 21.9489 22.895 22.0725 22.8831 22.202C22.8416 22.9202 22.711 23.6207 22.2422 24.2035C22.1294 24.3389 22.0404 24.5155 21.898 24.6038C21.5953 24.7922 21.5597 25.0571 21.5775 25.3632C21.5894 25.6104 21.5834 25.8518 21.5834 26.099C21.5775 26.4758 21.3579 26.6936 20.9721 26.7053C20.7822 26.7171 20.5923 26.7171 20.4084 26.7112Z"
							fill="#5A4DE3"
						/>
						<path
							d="M15.9302 14.0137C17.1171 14.0137 18.304 14.0255 19.491 14.0137C21.2417 13.9901 22.8797 15.4795 22.945 17.216C22.945 17.2631 22.9509 17.3161 22.9687 17.3632C23.0221 17.5339 22.9806 17.5987 22.7847 17.5692C22.0666 17.4515 21.3841 17.5869 20.7729 17.9695C19.9954 18.4522 19.5919 19.1704 19.5028 20.0711C19.491 20.177 19.4791 20.283 19.4613 20.389C19.3604 20.9482 19.3604 20.9482 18.8026 20.9482C16.7729 20.9482 14.7492 20.9482 12.7195 20.9482C12.3159 20.9482 12.3219 20.9482 12.2803 20.5538C12.2151 19.9298 12.1557 19.3058 11.7581 18.776C11.0815 17.8812 10.2151 17.4044 9.05781 17.5633C8.89164 17.5869 8.84416 17.5339 8.85009 17.3809C8.90351 16.4743 9.24772 15.6855 9.88273 15.0438C10.5059 14.414 11.2714 14.0372 12.1676 14.0196C13.4198 13.996 14.6779 14.0137 15.9302 14.0137Z"
							fill="#5A4DE3"
						/>
					</svg>

					Interior Unit
				</div>
				<button
					on:click={() => {
						$isInteriorUnitMinimized = !$isInteriorUnitMinimized;
					}}
					class="ghost-btn close-btn !px-0 !py-0"
					id="interior-toggle-amenities"
				>
					{#if !$isInteriorUnitMinimized}
						<img id="mm-am-btn" src={minimizeBtn} alt="minimize" />
					{/if}
					{#if $isInteriorUnitMinimized}
						<img id="mx-am-btn" src={maximizeBtn} alt="maximize" />
					{/if}
				</button>
			</div>

			<div class={!$isInteriorUnitMinimized ? 'block' : 'hidden'}>
				<div class="no-hovers pt-3">
					<div class="inner-btn-group">
						<Accordion.Root class="w-full sm:max-w-full">
							<Accordion.Item value="item-12" class="none">
								<Accordion.Trigger id="grolevel-fx">Top Level</Accordion.Trigger>
							</Accordion.Item>
							{#if $currentbhk === '4bhk'}
								{#each [{ id: '1-livingroom', label: 'Living Room' }, { id: '4-kitchen4bhk', label: 'Kitchen' }, { id: '7-masterbed4bhkph', label: 'Master Bedroom' }, { id: '9-masterwashroom', label: 'Masterbed Washroom' }, { id: '18-commonwashroom', label: 'Common Washroom' }, { id: '10-guestroom014bhk', label: 'Guest Room' }, { id: '12-guestwashroom1', label: 'Guestbed Washroom' }, { id: '13-kidroom', label: 'Kids Room' }, { id: '14-kidswashroom', label: 'Kids Washroom' }, { id: '2-deck', label: 'Deck' }, { id: '3-dinning', label: 'Dining Area' }] as scene}
									<button
										class={$hotspotName == scene.id ? 'active inner-modal-btn' : 'inner-modal-btn'}
										id={'am-' + scene.id}
										on:click={() => {
											$hotspotName = scene.id;
										}}
									>
										{scene.label}
									</button>
								{/each}
							{/if}
							{#if $currentbhk === '5bhk'}
								{#each [{ id: '1-living-room5bhk', label: 'Living Room' }, { id: '4-kitchen', label: 'Kitchen' }, { id: '10-masterbedroom', label: 'Master Bedroom' }, { id: '12-masterwashroom', label: 'Masterbed Washroom' }, { id: '13-guestroom1', label: 'Guest Bedroom 1' }, { id: '15-5bhkguestwashroom1', label: 'Guestbed Washroom 1' }, { id: '18-guestroom25bhk', label: 'Guest Bedroom 2' }, { id: '20-guestwashroom2', label: 'Guestbed Washroom 2' }, { id: '16-kidsroom', label: 'Kids Room' }, { id: '17-kidswashroom', label: 'Kids Washroom' }, { id: '3-deck5bhk', label: 'Deck' }, { id: '6-dinningarea', label: 'Dining Area' }, { id: '21-5bhkcommon-washroom', label: 'Common Washroom' }] as scene}
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
							{/if}
						</Accordion.Root>
					</div>
				</div>
			</div>
		</div>
	</div>
{/if}
<div id="pano"></div>
<div id="spot"></div>
