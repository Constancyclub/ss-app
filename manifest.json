const CACHE = 'ss-v3';
const ASSETS = ['/ss-app/', '/ss-app/index.html', '/ss-app/manifest.json'];

self.addEventListener('install', e => {
  e.waitUntil(caches.open(CACHE).then(c => c.addAll(ASSETS)));
  self.skipWaiting();
});

self.addEventListener('activate', e => {
  e.waitUntil(caches.keys().then(keys => Promise.all(keys.filter(k => k !== CACHE).map(k => caches.delete(k)))));
  self.clients.claim();
});

self.addEventListener('fetch', e => {
  e.respondWith(caches.match(e.request).then(r => r || fetch(e.request).catch(() => caches.match('/ss-app/index.html'))));
});

let scheduledTimers = [];
function clearTimers() { scheduledTimers.forEach(t => clearTimeout(t)); scheduledTimers = []; }

function msUntil(hour, minute) {
  const now = new Date();
  const target = new Date();
  target.setHours(hour, minute, 0, 0);
  if (target <= now) target.setDate(target.getDate() + 1);
  return target - now;
}

function msUntilNext(dayOfWeek, hour, minute) {
  const now = new Date();
  let diff = (dayOfWeek - now.getDay() + 7) % 7;
  if (diff === 0) {
    const todayTarget = new Date();
    todayTarget.setHours(hour, minute, 0, 0);
    if (todayTarget <= now) diff = 7;
  }
  const target = new Date();
  target.setDate(now.getDate() + diff);
  target.setHours(hour, minute, 0, 0);
  return target - now;
}

function parseTime(str) {
  const [h,m] = (str || '09:00').split(':').map(Number);
  return [h, m];
}

self.addEventListener('message', e => {
  if (!e.data || e.data.type !== 'SCHEDULE') return;
  clearTimers();
  const { settings, coldCount } = e.data;
  const [bh, bm] = parseTime(settings.blockTime);
  const [smh, smm] = parseTime(settings.storyManha || '08:30');
  const [sth, stm] = parseTime(settings.storyTarde || '17:00');

  if (settings.blocos !== false) {
    [1, 3, 5].forEach(day => {
      const delay = msUntilNext(day, bh, bm);
      scheduledTimers.push(setTimeout(() => {
        self.registration.showNotification('Bloco de SS — hora de prospectar', {
          body: 'Separa 1h: DMs, prospecção e leads frios.',
          icon: '/ss-app/icon.png'
        });
      }, delay));
    });
  }

  if (settings.frios !== false && coldCount > 0) {
    scheduledTimers.push(setTimeout(() => {
      self.registration.showNotification(`${coldCount} lead${coldCount>1?'s':''} frio${coldCount>1?'s':''}`, {
        body: 'Retoma o contato hoje antes de perder o timing.',
        icon: '/ss-app/icon.png'
      });
    }, msUntil(8, 0)));
  }

  if (settings.resumo !== false) {
    scheduledTimers.push(setTimeout(() => {
      self.registration.showNotification('Resumo semanal de SS', {
        body: 'Abre o app e confere como foi a semana.',
        icon: '/ss-app/icon.png'
      });
    }, msUntilNext(0, 19, 0)));
  }

  if (settings.storyManha !== false) {
    scheduledTimers.push(setTimeout(() => {
      self.registration.showNotification('Stories da manhã', {
        body: 'Hora de postar: rotina, dica, enquete ou treino.',
        icon: '/ss-app/icon.png'
      });
    }, msUntil(smh, smm)));
  }

  if (settings.storyTarde !== false) {
    scheduledTimers.push(setTimeout(() => {
      self.registration.showNotification('Stories da tarde', {
        body: 'Resultado, bastidor, CTA ou reflexão.',
        icon: '/ss-app/icon.png'
      });
    }, msUntil(sth, stm)));
  }
});
