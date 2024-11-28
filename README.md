# Functional-Programming-Project
import React, { useState } from 'react';
import { View, Text, TextInput, TouchableOpacity, FlatList, StyleSheet, Button, Alert } from 'react-native';
import * as Location from 'expo-location';

const App = () => {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const [selectedDay, setSelectedDay] = useState(new Date());
  const [attendance, setAttendance] = useState({});
  const [isLoggedIn, setIsLoggedIn] = useState(false);

  const subjects = [
    { name: 'Functional Programming', time: 'Senin, 08:40 - 10:20' },
    { name: 'Kecerdasan Buatan', time: 'Senin, 10:30 - 12:00' },
    { name: 'Struktur Data', time: 'Senin, 13:00 - 14:30' },
    { name: 'Statistika', time: 'Selasa, 15:00 - 16:30' },
    { name: 'MKWU', time: 'Rabu, 08:40 - 10:20' },
    { name: 'MKSI', time: 'Rabu, 10:30 - 12:00' },
    { name: 'Riset Operasi', time: 'Rabu, 13:00 - 14:30' },
    { name: 'Arsitektur Komputer', time: 'Kamis, 15:00 - 16:30' },
    { name: 'Grafika Komputer', time: 'Jumat, 14:00 - 15:20' },
    { name: 'Praktikum', time: 'Sabtu, 10.00 - 12.40' },
  ];

  const campusCoordinates = {
    latitude: -8.1675,
    longitude: 113.6843,
  };

  const getWeekDays = () => {
    const weekDays = [];
    const startOfWeek = new Date(selectedDay);
    startOfWeek.setDate(selectedDay.getDate() - selectedDay.getDay());

    for (let i = 0; i < 7; i++) {
      const date = new Date(startOfWeek);
      date.setDate(startOfWeek.getDate() + i);
      weekDays.push({ date: date.getDate(), fullDate: date });
    }
    return weekDays;
  };

  const getSubjectsForSelectedDay = () => {
    const dayOfWeek = selectedDay.getDay();
    const dayNames = ['Minggu', 'Senin', 'Selasa', 'Rabu', 'Kamis', 'Jumat', 'Sabtu'];
    return subjects.filter(subject => subject.time.includes(dayNames[dayOfWeek]));
  };

  const getDistanceFromLatLonInKm = (lat1, lon1, lat2, lon2) => {
    const R = 6371;
    const dLat = (lat2 - lat1) * Math.PI / 180;
    const dLon = (lon2 - lon1) * Math.PI / 180;
    const a = Math.sin(dLat / 2) * Math.sin(dLat / 2) +
              Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
              Math.sin(dLon / 2) * Math.sin(dLon / 2);
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
    const distance = R * c;
    return distance;
  };

  const markAttendance = async (subject) => {
    const currentTime = new Date();
    const [day, timeRange] = subject.time.split(', ');
    const [startTime, endTime] = timeRange.split(' - ');

    const [startHour, startMinute] = startTime.split(':').map(Number);
    const [endHour, endMinute] = endTime.split(':').map(Number);

    const startDateTime = new Date(selectedDay);
    startDateTime.setHours(startHour, startMinute, 0);

    const endDateTime = new Date(selectedDay);
    endDateTime.setHours(endHour, endMinute, 0);

    const selectedDateStr = selectedDay.toISOString().split('T')[0];
    const currentDateStr = currentTime.toISOString().split('T')[0];

    if (currentDateStr === selectedDateStr && currentTime >= startDateTime && currentTime <= endDateTime) {
      try {
        const { status } = await Location.requestForegroundPermissionsAsync();
        if (status !== 'granted') {
          Alert.alert("Izin ditolak", "Aplikasi memerlukan izin lokasi untuk absen.");
          return;
        }

        const userLocation = await Location.getCurrentPositionAsync({});
        const distance = getDistanceFromLatLonInKm(
          campusCoordinates.latitude,
          campusCoordinates.longitude,
          userLocation.coords.latitude,
          userLocation.coords.longitude
        );

        if (distance <= 2) {
          setAttendance((prevAttendance) => ({
            ...prevAttendance,
            [selectedDateStr]: {
              ...(prevAttendance[selectedDateStr] || {}),
              [subject.name]: {
                attended: true,
                time: currentTime.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
              },
            },
          }));
          Alert.alert(`Anda berhasil absen untuk ${subject.name} pada tanggal ${selectedDateStr}`);
        } else {
          Alert.alert("Tidak dapat absen", "Anda berada di luar jangkauan lokasi yang diizinkan.");
        }
      } catch (error) {
        console.error(error);
        Alert.alert("Error", "Terjadi kesalahan saat mendapatkan lokasi.");
      }
    } else {
      Alert.alert(`Anda tidak dapat absen. Jadwal untuk ${subject.name} adalah ${startTime} - ${endTime} pada ${selectedDay.toLocaleDateString('id-ID')}.`);
    }
  };

  const logout = () => {
    setIsLoggedIn(false);
  };

  const goToPreviousWeek = () => {
    const newDate = new Date(selectedDay);
    newDate.setDate(selectedDay.getDate() - 7);
    setSelectedDay(newDate);
  };

  const goToNextWeek = () => {
    const newDate = new Date(selectedDay);
    newDate.setDate(selectedDay.getDate() + 7);
    setSelectedDay(newDate);
  };

  return (
    <View style={styles.container}>
      {!isLoggedIn ? (
        <View style={styles.loginContainer}>
          <TextInput
            placeholder="Username"
            value={username}
            onChangeText={setUsername}
            style={styles.input}
          />
          <TextInput
            placeholder="Password"
            value={password}
            onChangeText={setPassword}
            secureTextEntry
            style={styles.input}
          />
          <Button title="Login" onPress={() => setIsLoggedIn(true)} />
        </View>
      ) : (
        <View style={styles.attendanceContainer}>
          <Text style={styles.selectedDay}>
            Hari/Tanggal: {selectedDay.toLocaleDateString('id-ID', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' })}
          </Text>

          <View style={styles.weekDaysContainer}>
            <TouchableOpacity onPress={goToPreviousWeek} style={styles.arrowButton}>
              <Text style={styles.arrowText}>⬅️</Text>
            </TouchableOpacity>
            {getWeekDays().map((day) => (
              <TouchableOpacity
                key={day.date}
                style={styles.dayButton}
                onPress={() => setSelectedDay(day.fullDate)}
              >
                <Text style={styles.dayText}>{day.date}</Text>
              </TouchableOpacity>
            ))}
            <TouchableOpacity onPress={goToNextWeek} style={styles.arrowButton}>
              <Text style={styles.arrowText}>➡️</Text>
            </TouchableOpacity>
          </View>

          <FlatList
            data={getSubjectsForSelectedDay()}
            keyExtractor={(item) => item.name}
            renderItem={({ item }) => {
              const selectedDateStr = selectedDay.toISOString().split('T')[0];
              const isAttended = attendance[selectedDateStr]?.[item.name]?.attended;
              const attendedTime = attendance[selectedDateStr]?.[item.name]?.time;

              return (
                <View style={styles.subjectContainer}>
                  <Text style={styles.subjectText}>{item.name}</Text>
                  <Text style={styles.scheduleText}>{item.time}</Text>
                  {isAttended ? (
                    <Text style={styles.markedText}>
                      Sudah Absen pada: {attendedTime}
                    </Text>
                  ) : (
                    <TouchableOpacity style={styles.attendanceButton} onPress={() => markAttendance(item)}>
                      <Text style={styles.buttonText}>Absen</Text>
                    </TouchableOpacity>
                  )}
                </View>
              );
            }}
            ListEmptyComponent={<Text style={styles.noDataText}>Tidak ada mata kuliah untuk hari ini.</Text>}
          />
          <Button title="Logout" onPress={logout} />
        </View>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  loginContainer: {
    justifyContent: 'center',
    alignItems: 'center',
    flex: 1,
  },
  input: {
    width: '80%',
    padding: 10,
    marginVertical: 10,
    borderWidth: 1,
    borderColor: '#ccc',
    borderRadius: 5,
  },
  attendanceContainer: {
    flex: 1,
  },
  selectedDay: {
    fontSize: 16,
    fontWeight: 'bold',
    marginVertical: 15,
    textAlign: 'center',
  },
  weekDaysContainer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 20,
  },
  arrowButton: {
    padding: 10,
  },
  arrowText: {
    fontSize: 20,
  },
  dayButton: {
    padding: 10,
    borderRadius: 5,
    backgroundColor: '#007BFF',
  },
  dayText: {
    color: '#fff',
    fontWeight: 'bold',
  },
  subjectContainer: {
    backgroundColor: '#fff',
    padding: 15,
    marginVertical: 5,
    borderRadius: 5,
    borderColor: '#ddd',
    borderWidth: 1,
  },
  subjectText: {
    fontSize: 16,
    fontWeight: 'bold',
  },
  scheduleText: {
    fontSize: 14,
    color: '#888',
    marginVertical: 5,
  },
  attendanceButton: {
    marginTop: 10,
    paddingVertical: 8,
    borderRadius: 5,
    backgroundColor: '#28a745',
  },
  buttonText: {
    color: '#fff',
    textAlign: 'center',
    fontWeight: 'bold',
  },
  markedText: {
    color: '#28a745',
    fontWeight: 'bold',
    marginTop: 10,
  },
  noDataText: {
    textAlign: 'center',
    color: '#888',
    fontStyle: 'italic',
  },
});

export default App;

