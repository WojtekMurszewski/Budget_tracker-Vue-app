<template>
    <div class="settings-page">
        <h2>Ustawienia</h2>
        <!-- Formularz ustawień powiadomień -->
        <form @submit.prevent="saveNotificationSettings">
            <!-- Pole formularza: Powiadomienia -->
            <label>
                <input type="checkbox" v-model="notificationsEnabled">
                Odbieraj powiadomienia
            </label>

            <!-- Przycisk zapisywania ustawień powiadomień -->
            <button type="submit">Zapisz ustawienia powiadomień</button>
        </form>

        <!-- Przycisk wylogowania -->
        <button @click="logout">Wyloguj</button>
    </div>
</template>

<script>
import { getAuth } from 'firebase/auth';
import { doc, getFirestore, setDoc, getDoc } from 'firebase/firestore';
import { defineComponent, ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import { getToken } from 'firebase/messaging';
import { messaging } from '../firebase/firebase';


export default defineComponent({
    name: 'SettingsPage',
    setup() {
        const notificationsEnabled = ref(false);
        const router = useRouter();
        const db = getFirestore();

        const fetchNotificationSettings = async () => {
            try {
                const auth = getAuth();
                const user = auth.currentUser;
                if (user) {
                    const userId = user.uid;
                    const userSettingsDocRef = doc(db, `userSettings/${userId}`);
                    const docSnap = await getDoc(userSettingsDocRef);
                    if (docSnap.exists()) {
                        notificationsEnabled.value = docSnap.data().notificationsEnabled;
                    }
                } else {
                    throw new Error('User not authenticated');
                }
            } catch (error) {
                console.error('Błąd podczas pobierania ustawień powiadomień:', error);
            }
        };

        const saveNotificationSettings = async () => {
            try {
                const auth = getAuth();
                const user = auth.currentUser;
                if (user) {
                const userId = user.uid;
                const userSettingsDocRef = doc(db, `userSettings/${userId}`);
                await setDoc(userSettingsDocRef, { notificationsEnabled: notificationsEnabled.value }, { merge: true });

                if (notificationsEnabled.value) {
                    try {
                        // Pobierz token FCM i zapisz go do Firestore
                        console.log('Uzyskiwanie tokenu FCM...');
                        const token = await getToken(messaging, { vapidKey: 'BPxGe-PhrQrXYah0TOYjmrahIUAoh5Sp8KdKWs5JqY1Yh_Jw_KlloOeOOkEzIwrj8jpAq6S6UZooXupZcRifGqY' });
                        console.log('Token FCM uzyskany:', token);
                        await setDoc(userSettingsDocRef, { fcmToken: token }, { merge: true });
                        console.log('Token FCM zapisany w Firestore');
                        } catch (tokenError) {
                        console.error('Błąd podczas uzyskiwania tokenu FCM:', tokenError);
                        throw new Error('Błąd podczas uzyskiwania tokenu FCM');
                        }
                    } else {
                        // Usuń token FCM, jeśli powiadomienia są wyłączone
                        await setDoc(userSettingsDocRef, { fcmToken: null }, { merge: true });
                    }

                alert('Ustawienia powiadomień zostały zapisane.');
                } else {
                throw new Error('User not authenticated');
                }
            } catch (error) {
                console.error('Błąd podczas zapisywania ustawień powiadomień:', error);
                alert('Wystąpił błąd podczas zapisywania ustawień powiadomień. Spróbuj ponownie później.');
            }
};


        const logout = async () => {
            try {
                // Wylogowanie użytkownika
                await getAuth().signOut(); // Wylogowanie z Firebase Authentication
                // Po wylogowaniu przekieruj użytkownika na stronę logowania
                router.push('/');
            } catch (error) {
                // Obsługa błędu wylogowania
                console.error('Błąd podczas wylogowywania użytkownika:', error);
                alert('Wystąpił błąd podczas wylogowywania użytkownika. Spróbuj ponownie później.');
            }
        };

        onMounted(() => {
            fetchNotificationSettings();
        });

        return {
            notificationsEnabled,
            saveNotificationSettings,
            logout
        };
    }
});
</script>

<style scoped>
.settings-page {
    text-align: center;
    padding: 20px;
}

button {
    display: block;
    margin: 10px auto;
    padding: 10px 20px;
    background-color: #f0f0f0;
    border: none;
    cursor: pointer;
}

button:hover {
    background-color: #e0e0e0;
}
</style>
